---
title: "A Trillion Parameters at a Thousand Tokens Per Second"
date: 2026-06-09T06:11:37+00:00
draft: false
slug: mimo-ultraspeed-trillion-parameter-inference
categories: [infrastructure]
tags: [inference, optimization, open-weights, quantization, speculative-decoding, moe]
params:
  author: AI Beat Desk
  summary: >-
    Xiaomi and TileRT published MiMo-V2.5-Pro-UltraSpeed on June 8, pushing
    a one-trillion-parameter model past 1000 tokens per second on a single
    standard 8-GPU node — no custom silicon, just three carefully chosen
    co-design decisions applied to a commodity cluster.
---

Getting a trillion-parameter model past 1000 tokens per second used to sound like a spec for specialized hardware. [MiMo-V2.5-Pro-UltraSpeed](https://mimo.xiaomi.com/blog/mimo-tilert-1000tps), released by Xiaomi and inference startup TileRT on June 8, reaches roughly 1200 tokens per second on a single standard 8-GPU node. No TPUs, no custom networking fabric — just a commodity cluster and three engineering decisions applied together.

The model itself is MiMo-V2.5-Pro, Xiaomi's trillion-parameter Mixture-of-Experts reasoning model. The UltraSpeed variant doesn't retrain it; it applies inference-time co-design that treats hardware bottlenecks as design constraints rather than facts of life.

The first decision is where to apply FP4 quantization. MoE models have a well-known bandwidth problem: the expert layers are large, scattered in memory, and activated sparsely, so memory bandwidth — not compute — is usually the limiting factor. The UltraSpeed approach quantizes only the MoE expert weights to FP4, leaving attention layers in full precision. That's a targeted choice: you take the precision hit where bandwidth dominates, avoid it where compute dominates, and keep the capability damage to a minimum.

The second decision is speculative decoding, but implemented differently than the standard autoregressive draft-verify loop. [DFlash](https://platform.xiaomimimo.com/docs/en-US/model-intro/mimo-v2.5-pro-ultraspeed) uses block-level masked parallel prediction — a small draft model generates a block of candidate tokens simultaneously rather than one at a time, using a sliding-window attention structure trained with the Muon optimizer. Acceptance length reaches 6.30 tokens per decode step on coding tasks and 5.56 on reasoning. That acceptance rate is high enough to matter, and the block generation means the draft phase doesn't serialize.

The third decision is kernel architecture. TileRT uses a persistent engine design where the inference pipeline stays resident on the GPU — no operator-boundary handoffs, no kernel launch overhead between attention and FFN. The warp specialization schedule runs at microsecond precision, keeping the compute pipeline full across the heterogeneous stages of a MoE forward pass.

Put together, the combination gets to roughly 10x the throughput of MiMo-V2.5-Pro's standard inference at approximately 3x the cost per token. For interactive coding or reasoning workloads where latency is load-bearing, that tradeoff is worth examining — you pay more per token but get a qualitatively different interactive feel.

The quantized weights and DFlash parameters are available on HuggingFace, which matters because the inference recipe is portable: the same FP4-on-experts + speculative decoding approach applies to other MoE models with similar architectures.

The broader point is less about MiMo specifically and more about what this demonstrates for the inference engineering space. The past two years have seen a pattern of hardware companies (NVIDIA with H200/B200, Google with TPU v6e) offering throughput gains that come bundled with hardware lock-in or steep infrastructure changes. The TileRT result pushes back on the assumption that thousand-tokens-per-second inference on trillion-parameter models is a hardware story. It's an engineering story. The cluster is the same; the design choices are different.

The access model is limited for now — application-only API trial through June 23 — but the published weights make it possible to replicate and iterate on the technique independently.
