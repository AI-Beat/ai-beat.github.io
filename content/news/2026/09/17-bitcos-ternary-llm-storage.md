---
title: "The Zeros in Your Ternary Model Are Wasted Space"
date: 2026-09-17T06:09:17+00:00
draft: false
slug: bitcos-ternary-llm-storage
categories: [inference]
tags: [inference, quantization, efficiency, hardware]
params:
  author: AI Beat Desk
  summary: >-
    Intel researchers surveyed 29 ternary LLM models and found zeros account for
    up to 51.5% of weights — which means the standard five-trit packing format
    silently wastes significant storage. BITCOS replaces it with a presence bitmap
    plus sign vector, reaching 1.485 bits per weight on sparse models and delivering
    1.10–1.28× end-to-end decode speedup on Intel CPUs and GPUs.
---

The "1.58 bits" in BitNet-b1.58 refers to \(\log_2 3\), the information-theoretic minimum needed to represent three values. Ternary LLMs store weights from the set \(\{-1, 0, +1\}\); in theory, you need 1.58 bits each. In practice, the standard storage format uses five-trit packing — three values in five bits, since \(3^3 = 27 < 32\) — which costs 1.667 bits per weight. Slightly above the theoretical minimum, but close enough that it hasn't attracted much attention.

[A paper from Intel researchers Evangelos Georganas, Alexander Heinecke, and Pradeep Dubey](https://arxiv.org/abs/2609.16338) points out that five-trit packing contains a hidden inefficiency: it assumes the three symbols are distributed roughly equally. They aren't. Across 29 ternary models, the authors measured zero density ranging up to 51.5% of all weights. The zeros dominate — and five-trit packing ignores this entirely.

Their proposed format, BITCOS, is straightforward once you see the distribution problem. It stores two things: a one-bit-per-weight presence bitmap indicating which weights are nonzero, then a one-bit-per-nonzero sign vector. Total cost per weight is \(2 - z\) bits, where \(z\) is zero density. At \(z = 0.515\), that's about 1.485 bits per weight. Five-trit is better than BITCOS only when zero density is below 37.5%, which holds for just 3 of the 29 models they tested.

The practical gains from better packing show up in matrix-vector multiplication, which is the dominant operation in autoregressive decode. Unpacking weights more efficiently means faster GEMV kernels. The paper provides implementations for AVX-512, AVX2, and Intel Xe2 GPUs — optimized unpacking sequences for each ISA, not pseudocode. On production kernels benchmarked across five platforms (Emerald Rapids and Arrow Lake CPUs, Arc 140V and Arc Pro B70 GPUs, and a fifth system), the GEMV speedup ranges from 1.01× to 1.28×. End-to-end decode throughput gains across seven ternary models land at 1.10–1.18× on CPUs and 1.09–1.27× on GPUs.

These are not transformative numbers. BITCOS won't make ternary models suddenly competitive with something else, and it doesn't change model quality in any way — the weights are unchanged, just stored differently. But context matters: in edge and on-device inference, where memory bandwidth is the binding constraint and every byte of model weight costs DRAM reads, a storage format that delivers 10-28% more throughput for free is meaningful. The format change happens at deployment time, not training time; a model trained with any ternary quantization scheme can use BITCOS storage without retraining.

The more interesting question is what happens as ternary models mature. Most quantization research in 2026 focuses on compressing existing float16/bfloat16 models post-hoc — GPTQ, AWQ, and their variants. BitNet-style native ternary training is a different approach, where the quantization constraint is baked into training from the start. If native ternary training continues to improve and closes the remaining quality gap with standard precision, the ecosystem around it — including storage formats, kernel implementations, hardware support — starts to matter a lot. BITCOS is early infrastructure for that ecosystem.

The Intel authorship is worth noting. Intel has obvious commercial interest in efficient inference on its CPU and Arc GPU lines, and this paper delivers validated implementations for those specific platforms. It's applied research with a clear deployment target, which is usually a good sign for correctness and practicality. The code, to the extent it's referenced, maps directly to production-deployable implementations rather than research prototypes.
