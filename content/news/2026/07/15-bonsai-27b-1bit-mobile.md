---
title: "A 27B Model in 3.9 Gigabytes"
date: 2026-07-15T06:11:53+00:00
draft: false
slug: bonsai-27b-1bit-mobile
categories: [inference]
tags: [inference, quantization, on-device, mobile, open-source, qwen]
params:
  author: AI Beat Desk
  summary: >-
    PrismML released Bonsai 27B on July 14: 1-bit binary and ternary builds of
    Qwen3.6-27B that fit in 3.9 GB and 5.9 GB respectively, run at 11 tok/s on
    an iPhone 17 Pro, and retain over 90% and 95% of full-precision benchmark
    performance. The compression factor is around 14× versus FP16, and the models
    are available under Apache 2.0.
---

PrismML [released Bonsai 27B](https://prismml.com/news/prismml-releases-bonsai-27b) on July 14: two extreme-compression versions of Qwen3.6-27B, in 1-bit binary and 1.58-bit ternary quantization, available under Apache 2.0. The 1-bit version weighs 3.9 GB and runs at 11 tokens per second on an iPhone 17 Pro. The ternary version is 5.9 GB, optimized for laptops, and hits 87 tok/s on an M5 Max.

The parameter count is 27.8 billion. In standard FP16, that would require roughly 55 GB of memory — more than any phone, and more than most consumer GPU configurations. What makes either variant fit on a phone is a constraint baked into training: rather than learning free-floating floating-point weights, the model learns weights restricted to either {-1, +1} (binary, the "1-bit" case) or {-1, 0, +1} (ternary, requiring 1.58 bits per parameter). Both representations are radically cheaper to store and operate on than FP16, and both can be computed efficiently with custom SIMD operations that do additions and subtractions instead of multiplications.

The benchmark retention numbers matter more than the compression factor alone. A model that compresses 14× but loses half its capability on reasoning tasks is not useful. PrismML reports that the 1-bit variant retains over 90% of full-precision benchmark performance, and the ternary variant retains over 95%. These aren't broken down by task in the announcement, which is worth flagging — aggregate benchmark retention can hide significant degradation in specific areas like multi-step reasoning or long-context retrieval. The whitepaper has more detail.

The training setup is notable: models were trained on Google v5 TPUs. BitNet-style 1-bit quantization is typically done with quantization-aware training (QAT), not post-training quantization — you constrain the weights *during* training so the model learns to operate within those constraints rather than having its learned weights approximated afterward. This is substantially harder to set up than PTQ, which partly explains why usable 1-bit quantized models at the 27B scale haven't been common until recently.

At 11 tok/s on an iPhone 17 Pro, the 1-bit variant is genuinely usable for interactive inference. Most people read faster than they type, but 11 tok/s is roughly the speed of a reasonably fast human reader — fast enough that generation won't feel like watching paint dry. The ternary variant on a MacBook (with an M-series chip's unified memory making the 5.9 GB weight fit cleanly) runs faster than many hosted inference endpoints under load.

The practical implication is what it's been for prior on-device model work, but at a size class that changes what's plausible: agentic and reasoning-capable workflows that don't require a network connection, don't send code or documents to a third-party API, and don't accumulate inference costs at scale. The model supports multimodal input (text and image), tool calling, and agentic workflows per the announcement. Whether those capabilities hold up at 90-95% benchmark retention compared to the full-precision base model is the open question, but the numbers are sufficient to take seriously.

The underlying base, [Qwen3.6-27B](https://huggingface.co/Qwen/Qwen3.6-27B), was itself released earlier this year as a strong general-purpose open-weights model. PrismML's contribution is the compression infrastructure and the TPU training pipeline that makes the quantized variants work at this scale. The Apache 2.0 license on both Bonsai variants, combined with the Qwen base license, means these can be built on commercially without restrictions.

Quantization at this level a few years ago would produce something you could run on a phone but wouldn't necessarily want to use for real work. The gap has closed considerably.
