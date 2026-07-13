---
title: "Open Kernels for Sparse Attention Training"
date: 2026-07-13T06:12:26+00:00
draft: false
slug: flash-msa-sparse-attention-training
categories: [open-source]
tags: [open-source, attention, long-context, training, kernels, cuda, minimax]
params:
  author: AI Beat Desk
  summary: >-
    Flash-MSA, published July 11, provides the first open-source performant
    training kernels for MiniMax Sparse Attention — the block-sparse attention
    mechanism that enabled M3's 28.4× compute reduction at 1M context. The
    CuTeDSL implementation targets Hopper and Blackwell GPUs and adds
    group-specialized proxy heads, making sparse-attention training accessible
    outside of frontier lab infrastructure.
---

When [MiniMax M3](https://www.minimax.io/blog/minimax-m3) launched in June, the headline architecture was MiniMax Sparse Attention (MSA). The [MSA paper on arxiv](https://arxiv.org/abs/2606.13392) described a block-partitioned sparse attention mechanism — factored into an Index Branch that selects which KV blocks each query reads, and a Main Branch that runs exact softmax attention over only those blocks. At 1M context, this cuts per-token attention compute to about 1/28th of full attention while matching dense quality on downstream tasks. MiniMax's co-designed CUDA kernel achieved 14.2× prefill and 7.6× decoding speedups on H800 GPUs.

That was the inference side of the story. Training with sparse attention is a different problem. The forward pass with block-selection is manageable, but backpropagating through a learned sparse pattern requires custom kernels that handle gradient flow through the gating mechanism. No open-source training kernels for MSA existed until last Friday, when a developer named Ganesh Nanduru published [Flash-MSA](https://nanduruganesh.github.io/flash-msa/).

The implementation is written in [CuTeDSL](https://github.com/NVIDIA/cutlass/tree/main/python/cutlass/cute/dsl), NVIDIA's Python-embedded DSL for CUTLASS-style tiled matrix math on Hopper (H100) and Blackwell (B200) GPUs. CuTeDSL is relatively new — it was designed to make writing kernel-level code less painful than raw CUDA while preserving the tile-level control needed for custom attention patterns. The Flash-MSA implementation uses 128-token blocks for the sparsity granularity, which matches MiniMax's approach.

The technical addition beyond a straight port of MSA is the group-specialized proxy head design. In standard MSA, an Index Branch head selects which KV blocks a query sees. In Flash-MSA, different subsets of attention heads each maintain their own proxy head that selects different KV blocks — so a head in the first group might specialize on recent tokens while a head in another group attends to distant context. The proxy heads are lightweight (small trained networks that output block selection scores), and their specialization emerges from the training objective rather than being designed in. Whether this translates to better model quality at a fixed parameter budget is something that requires running actual training runs, but the mechanism is plausible: heterogeneous attention patterns have been useful in other contexts (mixture-of-depths, sliding window combined with full attention, etc.).

The benchmark numbers in the repository show Flash-MSA's training throughput beating standard FlashAttention across sequence lengths, with the gap growing as sequences get longer — which is the expected behavior for sparse attention: at short sequences the overhead of block indexing washes out the savings from skipping computation, but at 512K+ tokens the compute reduction dominates.

The context here matters. MiniMax's inference kernels were specific to their deployment stack. GLM-5.2, which [shipped with open MIT weights in June](/news/2026/06/glm-5-2-weights-delivered/) and uses a related IndexShare sparse attention mechanism, provided weights and inference code but not the training machinery. If you wanted to train a model from scratch using block-sparse attention patterns similar to what M3 and GLM-5.2 use, you were building kernels from scratch or working from the MSA paper with CUDA.

Flash-MSA changes that calculus for teams with Hopper or Blackwell hardware. The implementation is a single-developer project, not a lab release, so the caveats apply: it hasn't been audited for numerical correctness under all gradient accumulation scenarios, and Blackwell support is listed as in-progress in the repository. But the core Hopper training path is benchmarked and documented, the code is readable, and the contribution fills a real gap in the open-source long-context training toolkit.

The broader trend is that sparse attention techniques — which frontier labs developed to make million-token context tractable — are now becoming accessible to independent researchers and smaller teams. Flash-MSA is one piece of that; DeepSeek V4 Pro's DSA mechanism from April was another, as was the IndexShare approach in GLM-5.2. The component that remains least accessible is training *data* at the scale required to make these architectures useful, but the kernel gap is narrowing.
