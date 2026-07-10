---
title: "Tencent's Hy3: Apache-Licensed and Punching Above Its Weight"
date: 2026-07-10T06:12:16+00:00
draft: false
slug: hy3-tencent-open-moe
categories: [models]
tags: [models, open-source, moe, reasoning, open-weight]
params:
  author: AI Beat Desk
  summary: >-
    Tencent released Hy3 on July 6 under Apache 2.0 — a 295B MoE model with
    21B active parameters that scores 90.4 on GPQA Diamond and 78.0 on
    SWE-Bench Verified, matching or exceeding models two to five times its
    active-parameter count. It's available for free on OpenRouter through
    July 21 and on Hugging Face in both full FP16 and FP8 quantized forms.
---

Tencent's [Hy3 release](https://www.tencent.com/en-us/articles/2202386.html) on July 6 is another entry in the now-crowded field of Chinese open-weight MoE frontier models, but it's worth looking at the benchmark numbers because they're genuinely interesting.

Hy3 has 295B total parameters and 21B active per token — the same sparse-routing architecture that makes GLM-5.2 and Kimi K2 efficient at inference time. On GPQA Diamond (a graduate-level science reasoning benchmark that's hard to saturate), it scores 90.4, compared to GPT-5.5's 93.6. On SWE-Bench Verified it hits 78.0. On HLE-with-tools, 53.2. The model also scored 90.0 on IMOAnswerBench and 72.0 on USAMO 2026 — both targeting mathematical competition problems where pure reasoning matters more than recall.

For context, those GPQA and SWE-bench numbers put Hy3 in the same tier as models reporting many times its active parameter count. That's consistent with the broader pattern: MoE models are trained on enormous capacity (295B total params worth of representational space) but serve inference at a fraction of that cost. The training budget buys you a capable model; the routing mechanism buys you cheap inference.

The license is Apache 2.0 — permissive commercial use, no fine-tuning restrictions. The full model is available on [Hugging Face](https://huggingface.co/tencent/Hy3-preview) in a 598 GB FP16 version and a 300 GB FP8 quantized version. It's also on OpenRouter for free through July 21, after which it presumably moves to paid tiers.

The architecture uses a sparse MoE with 192 experts per layer and top-8 routing — so 8 of 192 experts fire per token per MoE layer. The context length is 256K. [Simon Willison tested it](https://simonwillison.net/2026/jul/6/hy3/) against creative prompts and found it capable enough; an earlier preview version had briefly topped OpenRouter's model rankings by usage margin.

One thing worth noting about this release: the [VentureBeat comparison](https://venturebeat.com/technology/tencents-apache-licensed-hy3-takes-on-glm-5-2-at-half-the-size-and-wins-everywhere-except-coding) between Hy3 (295B/21B) and GLM-5.2 (744B/40B) found Hy3 winning on most tasks except coding, at roughly half the active-parameter count. That's a useful data point about the current state of MoE scaling: you don't necessarily get better coding just by routing more parameters per token. The coding gap between Hy3 and GLM-5.2 is one of the few places where the larger model's additional activated capacity makes a measurable difference.

For teams that want a reasoning-capable open-weight model they can run and fine-tune commercially, Hy3 is a reasonable option in July 2026. It's not clearly ahead of GLM-5.2 or Kimi K2.7 Code across the board, but the Apache license, free API access for the next two weeks, and 295B/21B active ratio make it worth evaluating if you're in the market for something in this class.
