---
title: "The Rest of the Transformer, Fused"
date: 2026-05-22T06:09:05Z
draft: false
slug: coda-gemm-epilogue-transformer
categories: [inference]
tags: [kernels, gpu, transformers, training, inference, FlashAttention]
params:
  author: AI Beat Desk
  summary: >-
    CODA, a new paper from Tri Dao and colleagues, extends FlashAttention's core
    insight — keep data on-chip, avoid DRAM round-trips — to all the non-attention
    operations in a transformer block. Norms, activations, residuals, and projections
    are reparameterized as GEMM epilogues so they run while output tiles are still
    in SRAM. It's a surgical attack on the memory wall that's been hiding in plain
    sight since FlashAttention fixed attention.
---

There's a well-known efficiency story about transformers: attention was the bottleneck, FlashAttention fixed it by tiling the computation and keeping intermediate values in SRAM rather than rounding them through global memory, and that was that. What the story glosses over is that attention is not the whole model. Everything else — layer norms, activation functions, residual additions, projection scaling — kept running as ordinary memory-bound kernels. The GEMM is compute-bound and fast; the surrounding ops are memory-bound and slow; and every time you write results to DRAM only to read them back for the next fused kernel, you're burning bandwidth for no arithmetic reason.

[CODA](https://arxiv.org/abs/2605.19269), submitted to arXiv this week by Han Guo, Jack Zhang, Arjun Menon, Driss Guessous, Vijay Thakkar, Yoon Kim, and Tri Dao, addresses this directly. The core claim is that the entire transformer block can be reformulated as a sequence of GEMM-plus-epilogue programs, where the epilogue runs while the GEMM output tile is still on-chip — before it gets written to global memory. Normalization, activation, residual updates, tensor reductions: these ops are "algebraically reparameterized" so they can be expressed as composable epilogue primitives sitting inside the matmul's shadow.

The analogy to FlashAttention is not accidental. FlashAttention's key contribution was showing that you could restructure attention to never materialize the full \(N \times N\) attention matrix in HBM. CODA applies the same intuition to what was left behind: if you're going to write a GEMM output tile to memory anyway, can you also do the normalization and activation before you write it? If yes, you've just eliminated a kernel launch, a global memory read, and a global memory write — three sources of latency per layer, multiplied across every block in every model.

The implementation is built on [CUTLASS CuTeDSL](https://github.com/NVIDIA/cutlass) and targets NVIDIA Hopper (H100). The code is available at [HanGuo97/coda-kernels](https://github.com/HanGuo97/coda-kernels), which includes both human-authored kernels and, interestingly, LLM-authored implementations — the paper evaluates both, finding that "both human- and LLM-authored CODA kernels achieve high performance" on representative transformer workloads. That's worth noting: kernel programming via CuTe is notoriously difficult, and if the pattern is regular enough that an LLM can produce correct, high-performance implementations, that has implications for how quickly this technique gets adopted.

The operations CODA targets span both the forward and backward pass: not just norms and activations but also the backward reductions that are often the ugliest part of custom training kernels. One of the perennial pains of writing efficient training code is that the backward pass requires gradients of things like RMSNorm to flow through without forcing additional memory allocations, and the epilogue fusion approach handles this by construction.

It's worth placing this in context. FlashAttention became nearly universal because the performance gain was large, the interface was clean, and it dropped into existing frameworks without requiring architecture changes. CODA has the same structural properties: it doesn't change what the model computes, only how the computation is organized on hardware. The implementation complexity is real — CuTeDSL is not a beginner tool — but the abstraction it provides lets you write "speed-of-light kernels for transformer operations" with composable primitives rather than hand-rolling each fused kernel from scratch.

Whether CODA gets the same traction as FlashAttention depends on how well it ports to other hardware and how much friction there is in integrating it with PyTorch compilation paths. The current release is H100-only, which covers most serious training clusters but excludes the long tail of hardware. Still, the pattern itself — fuse everything into the epilogue, not just the obvious thing — is clearly correct, and the question is just when it becomes standard practice rather than a research result.

Also released this week: [KVBoost](https://pythongiant.github.io/KVBoost/), a drop-in HuggingFace wrapper that hashes prompt chunks and reuses cached KV pairs across requests wherever prefixes match. On a ShareGPT multi-turn replay it held time-to-first-token flat at ~20ms through eight turns versus a baseline that grew to 122ms — about a 4.6× improvement at turn eight with no measurable accuracy loss. Chunk-level prefix reuse isn't new (vLLM's prefix caching does something similar) but the HuggingFace-native API and the cross-request sharing make it more accessible for single-machine serving.
