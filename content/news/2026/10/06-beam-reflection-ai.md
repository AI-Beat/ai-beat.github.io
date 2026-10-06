---
title: "After Two Years of Hype, Reflection Ships"
date: 2026-10-06T06:12:07+00:00
draft: false
slug: beam-reflection-ai
categories: [models]
tags: [open-weights, moe, rl, inference]
params:
  author: AI Beat Desk
  summary: >-
    Reflection AI finally announces its first model, Beam — a 501B MoE with 23B
    active parameters and a technically interesting RL training story. The weights
    aren't out yet, but the benchmark numbers and the compute details make a case
    that the lab has been building something real.
---

Reflection AI has been a recurring story in the AI press for two years without shipping a model. Founded by Misha Laskin and Ioannis Antonoglou (an AlphaGo author), the company raised north of $4 billion on a pitch to be "America's open frontier lab" — the Western answer to DeepSeek. Timelines slipped from early 2026 to "later this year." The jokes wrote themselves.

Yesterday they published [the announcement for Beam](https://reflection.ai/blog/introducing-beam): a 501 billion parameter sparse mixture-of-experts model with 23 billion parameters active per token, a 1 million token context window, targeting coding, reasoning, and agentic workloads.

The weights are not out yet — the blog post promises them "later this month" along with a technical report. So this is still an announcement, not a release. Given Reflection's track record with timelines, a degree of skepticism is warranted. But unlike prior "coming soon" communications, this one comes with actual numbers.

## What the spec sheet says

The architecture — large total parameter count, small active fraction per token — is familiar. Qwen3.8-Flash-Next, Kolibri-1, DeepSeek V4.1 Flash all play in this space. MoE at 501B total with ~4.6% activation ratio is aggressive even by current standards; the routing overhead and load balancing challenges scale with the number of experts, and Reflection hasn't said how many experts the model uses. The 1M token context suggests they've done something real on the attention side, though that number alone is becoming table stakes.

The pretraining story is unremarkable: 23.8 trillion tokens on 6,144 NVIDIA GB300 NVL72 GPUs in under four weeks. With enough hardware, you can train a lot of things. What's more interesting is the reinforcement learning phase.

Reflection reports over 100 million rollouts across roughly 1.3 billion sandbox environments on 10,500 GB300 GPUs over four weeks. That's a lot of RL. For context: a million rollouts is considered substantial for a research paper; getting to 100 million at this scale implies either very short episodes or a massive parallelism setup that can sustain dense reward signal continuously. The coding and agentic benchmark results — AIME 2026 at 97.8, SWE-Bench Pro v2-Hard at 77.2, Terminal Bench v2.1 at 80.1 — are consistent with a model that has been subjected to heavy RL on verifiable tasks, not just SFT on demonstrations.

## What to make of it

The open-weight angle is the one thing that's hard to fake. If the weights ship Apache 2.0 as promised, Reflection delivers on the actual commitment that justified the fundraising narrative: weights you can run, fine-tune, and inspect without a cloud dependency. Apache 2.0 with no use-restriction rider would put Beam among the most accessible frontier-class models available, alongside DeepSeek V4.1 Flash (748B total, MIT) and Qwen3.8-Flash-Next.

The catch is that "announcement with weights coming later" is exactly what a lab would say if they wanted to lock in the news cycle without committing to a date. The technical report, when it arrives, will be the thing to read: expert count, routing strategy, the RL curriculum, why 1.3B distinct sandbox environments rather than a smaller resampled set. Those details either exist or they don't, and they'll determine whether the benchmark numbers reflect genuine reasoning capability or careful post-training on benchmark-adjacent distributions.

For now, Beam is the most technically credible thing Reflection has put in public. Whether the weights follow — and what the technical report actually says — is the story to watch for the rest of October.
