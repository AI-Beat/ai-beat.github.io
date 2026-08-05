---
title: "Born Ternary"
date: 2026-08-05T06:15:00+00:00
draft: false
slug: maple-preview-ternary-moe
categories: [inference]
tags: [inference, apple, moe, open-source, on-device, ternary, reasoning]
params:
  author: AI Beat Desk
  summary: >-
    DeepGrove's Maple-Preview is a 20B ternary-weight MoE reasoning model trained
    from scratch at {-1, 0, +1} precision — not quantized down from float. The
    5.31 GB checkpoint runs at 218 tokens/second on an M4 Mac mini and 120 tok/s
    on iPhone, with competitive AIME and GPQA-D scores and an MIT license.
---

The standard way to make a large model small is quantization: train at full precision, then crush the weights into lower-bit formats after the fact. It works, but there's a structural tension — the model spent training learning to use float32 or bfloat16 values, and you're forcing it to approximate those values with far fewer bits once the training is over.

[Maple-Preview](https://huggingface.co/deepgrove/maple-preview), released today by DeepGrove, takes the other route. Each weight is constrained to the ternary set {-1, 0, +1} throughout training, not imposed afterward. That's roughly 1.58 bits of information per parameter — below INT2, and substantially below the 4-bit quantization most optimized runtimes use. The optimizer never has access to higher-precision weights to begin with, which forces it to find solutions that are naturally representable in ternary rather than approximations of solutions that weren't.

Stack that on a Mixture-of-Experts architecture — 24 layers, 256 experts, 8 active per token — and the effective parameter count per forward pass is around 1 billion out of 20 billion total. The resulting checkpoint is 5.31 GB. On an M4 Mac mini, Maple-Preview runs at 218 tokens per second. On an iPhone, it reaches 120 tokens per second.

To put those numbers in context: the [Swiftlet](https://github.com/leonickson1/Swiftlet) runtime covered here yesterday achieved 4.5–5 tok/s for an 80B model by streaming expert weights from SSD on demand. Maple-Preview's approach is structurally different — it's not about smarter IO; it's about making the model itself smaller at the representation level, so it fits into memory without special tricks. Swiftlet and Maple-Preview aren't competing solutions so much as complementary ones: one optimizes the access pattern, the other optimizes the weight density.

The attention layout is a 3:1 ratio of sliding-window attention (SWA-512) to global attention layers, with a 131K token context. Sliding-window attention caps the per-token attention span to 512 positions in most layers, which significantly reduces the memory cost of the KV cache over long contexts; the periodic global attention layers let the model attend across the full sequence when needed. This is a well-established design pattern for long-context efficiency, and it's sensible here given the memory constraints.

The benchmark claims are the more surprising part. The model reports competitive scores on AIME 2026 and HMMT 2026 — math competitions that test hard problem-solving, not pattern matching — as well as GPQA-D, which evaluates graduate-level science reasoning. DeepGrove describes it as a new point on the Pareto frontier for memory-to-performance and speed-to-performance in its weight class.

The model card is honest about the current state: Maple-Preview "received minimal post-training for agentic tasks and only small-scale general reinforcement learning." The Hacker News comments surfaced this quickly — the model is prone to confident-sounding but factually incorrect responses on knowledge retrieval tasks. The reasoning scaffolding is there; the general-purpose reliability isn't fully developed yet. This is a preview.

DeepGrove previously published [Bonsai](https://github.com/deepgrove-ai/Bonsai), a ternary 0.5B model trained from scratch earlier in 2026. Maple-Preview applies the same training methodology at 40× the scale, layered with MoE. The fact that the approach is scaling is the noteworthy result — ternary training isn't a hack that works only for tiny models. The [Spectra](https://arxiv.org/abs/2407.12327) paper from 2024 was an early demonstration of this, showing that ternary LLMs trained from scratch can be surprisingly competitive at scale; DeepGrove is extending that finding into MoE territory.

MIT license, weights on Hugging Face. A ternary-weight MoE with competitive reasoning benchmarks, in 5.31 GB, running at native speed on consumer Apple hardware, is a reasonable base for experimentation. The "preview" label signals that the interesting part — post-training on agentic tasks, broader RL — is still ahead.
