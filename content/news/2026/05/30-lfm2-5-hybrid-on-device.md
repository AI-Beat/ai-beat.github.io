---
title: "Liquid AI's LFM2.5: When Half Your Layers Aren't Attention"
date: 2026-05-30T06:09:00+00:00
draft: false
slug: lfm2-5-hybrid-on-device
categories: [models]
tags: [models, architecture, inference, edge]
params:
  author: AI Beat Desk
  summary: >-
    Liquid AI ships LFM2.5-8B-A1B, a 38T-token trained hybrid model where 18
    of 24 layers are gated convolution blocks rather than attention — and it
    reaches 253 tokens/second on an M5 Max CPU with under 6 GB of memory.
---

Most 8B models in 2026 look roughly the same at the architecture level: a stack of transformer blocks with grouped-query attention, a big feedforward layer, RoPE embeddings, and some variant of flash attention. [Liquid AI's LFM2.5-8B-A1B](https://www.liquid.ai/blog/lfm2-5-8b-a1b), released May 28, does something different. Of its 24 layers, 18 are double-gated LIV convolution blocks — not attention — and only 6 use GQA. The model is a sparse mixture of experts, activating roughly 1.5B of its 8.3B total parameters per forward pass. This is the same hybrid architecture lineage Liquid AI has been developing since 2024, now scaled up considerably.

The training numbers are notable. The previous LFM2-8B-A1B was trained on 12 trillion tokens. LFM2.5 was trained on 38 trillion, with a two-stage context extension: a 2T-token phase to reach 32K context, then an additional 400B tokens to extend to 128K. That's a lot of compute for a model this size, and the coverage shows — it's not just benchmark-padding, since the multilingual tokenizer improvements (120% efficiency gain for Hindi, 238% for Thai) suggest real vocabulary and data diversity was added.

The benchmarks are competitive. IFEval comes in at 91.84, slightly ahead of Gemma-4-26B at 91.40. MATH500 hits 88.76, AIME25 at 42.53. The AA-Omniscience non-hallucination index is 63.47%, which the blog post contrasts with competitors in the low single digits. Agentic tool-calling benchmarks (BFCLv3 at 64.36) are solid if not dominant.

What's most interesting is the efficiency profile. On an Apple M5 Max, the model runs at 253 tokens per second decoding under 6 GB of memory. On a Ryzen AI Max+ 395 that's 146 tokens/sec. An iPhone is around 30 tokens/sec. These are CPU inference numbers, not GPU — the kind of performance that matters if you're building something that runs locally without cloud dependency. On the server side, a single H100 can handle 18,500 output tokens per second at high concurrency and sustains over 1.6 billion tokens per day. The sparse MoE design is doing real work here: you're only running 1.5B active parameters per token despite having 8.3B available, which keeps memory bandwidth requirements low.

LFM2.5 is reasoning-only — it always produces a chain of thought before its final answer, which is a deliberate design choice rather than an option. Liquid AI has also published [llama.cpp, MLX, vLLM, and SGLang](https://huggingface.co/LiquidAI) support from day one, and the model is available under an open-weight license with unrestricted deployment. No commercial use restrictions.

The interesting question this model raises is why the rest of the field hasn't moved further on non-attention architectures. State-space models and hybrid convolution-attention designs have been compelling in research for a couple of years, and Liquid AI's repeated shipping of production-quality hybrids is one of the few consistent data points that the departure from pure transformers can actually work in practice, not just in ablations. The LFM2.5 release doesn't resolve the architectural debate, but it moves the weight of evidence a little further toward "the alternatives are real."
