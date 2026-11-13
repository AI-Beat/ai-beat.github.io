---
title: "The Ghost in the Weights"
date: 2026-11-13T06:14:00+00:00
draft: false
slug: gpu-fingerprint-chinese-models
categories: [policy]
tags: [export-controls, nvidia, deepseek, forensics, hardware]
params:
  author: AI Beat Desk
  summary: >-
    Unemployment Labs published a forensic analysis showing that six major Chinese
    AI models — including DeepSeek-V4-Pro, Kimi K3, and MiniMax-M3 — carry
    statistical signatures in their weights that match Nvidia's Blackwell GPU
    architecture, which is currently banned from export to China. The method is
    technically elegant, legally limited, and politically timely.
---

When you initialize a large model, you do not start from zero. The standard approach — Kaiming or Xavier initialization, or one of several variants — fills weight matrices with pseudorandom numbers drawn from a seeded generator. That generator runs in parallel across GPU threads, and because each GPU architecture defines a specific number of streaming multiprocessors, the resulting layout of random values is architecture-specific. A Blackwell B300 with 148 SMs spreads its random draws differently than an H100 with 132. These initialization values get overwritten quickly once training begins, but not entirely: faint traces survive in the final published weights as a kind of hardware signature.

This is the core insight behind [a report published November 12](https://unemploymentlabs.com/checkpoint-forensics) by Will DePue, an OpenAI researcher who worked on Sora and GPT-5 post-training, writing under the name Unemployment Labs. DePue searched over six trillion combinations of hardware layout and random seed against the published weights of recent Chinese AI models. What he found is striking: six models from DeepSeek, Moonshot AI, MiniMax, and Ant Group carry signatures consistent with Nvidia's Blackwell architecture — hardware that the US Commerce Department bars from export to China.

The six models break down by Blackwell variant. [DeepSeek-V4-Flash](https://www.deepseek.com) and MiniMax-M3 match the B200/B300 layout (148 SMs). [DeepSeek-V4-Pro](https://www.deepseek.com), [Kimi K3](https://kimi.moonshot.cn), Kimi K3.1-Pro, and Ant Group's Ling-3.1-flash match the GB200/GB300 layout (152 SMs). Earlier models from the same companies matched Nvidia's older Hopper generation instead, which is consistent with a hardware upgrade at some point in 2025 or early 2026. Models from Qwen, GLM, Tencent, Xiaomi, and StepFun showed Hopper signatures; one model left no Nvidia fingerprint at all.

The statistical confidence numbers are hard to dismiss. The weakest signal in any single test came in at 20 sigma above chance; the full-model matches ranged from 49 to 1,385 sigma. DePue reports that the best accidental match he found in his search was around 8 sigma — meaning the gap between noise and signal is not marginal. He also validated the method against 24 models whose developers had publicly disclosed their training hardware, and found agreement in every case.

What the method cannot tell you is where the chips were or how the companies got access to them. The initialization fingerprint identifies the hardware that produced the starting weights, not every GPU used in training, and not the physical location of those GPUs. Access via third-party data centers in Malaysia or Thailand — the route that a [September 2026 CNAS report](https://www.cnas.org/publications/cnas-insights/cnas-insights-america-must-expand-not-weaken-ai-export-controls-at-upcoming-talks-with-china) described as a gray area under current Commerce rules — would leave the same signature as chips physically smuggled into China. So would legitimate access through a loophole. The analysis does not, by itself, establish that any export rule was violated.

That caveat matters, but it does not make the analysis uninteresting. The technical contribution here is separable from the policy claim. DePue has essentially built a hardware-attribution primitive: given open-weights model files, you can often identify the chip family used at initialization. As a forensic technique, it works remarkably well on the 24 ground-truth cases.

The political context gives the report its edge. The Trump administration official cited by Reuters earlier this year claimed DeepSeek was training on Blackwell hardware and planned to "scrub technical fingerprints" before releasing the weights. DePue's report suggests either the scrubbing did not happen or is not straightforward to do. Moonshot AI, which builds Kimi, was specifically named in [White House claims](https://www.cnas.org/publications/cnas-insights/cnas-insights-america-must-expand-not-weaken-ai-export-controls-at-upcoming-talks-with-china) about cluster access in Thailand and reportedly sought access to 8,000-GPU Blackwell configurations in Europe.

Nvidia has in parallel been testing optional [location-verification software](https://www.techloy.com/nvidia-is-testing-software-that-verifies-the-location-of-its-blackwell-ai-chips.md) that uses GPU telemetry and network-latency attestation to infer where chips are running. The company frames this as fleet management rather than export enforcement. DePue's method is different and complementary: it works post-hoc on released model files, requires no cooperation from the hardware owner, and needs no special software installed.

The report has not been peer-reviewed and comes from one researcher at a small nonprofit. But the methodology is explained clearly enough to invite independent replication — anyone with access to a physical Blackwell system and the released model weights can run the seed-recovery check that DePue describes. That is how this kind of claim earns credibility, and the next few weeks will likely test it. If the matches hold up under scrutiny, weight-initialization forensics will become a standard tool in the already-contentious debate over whether export controls are working.
