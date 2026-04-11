---
title: "Thirty People, Four Hundred Billion Parameters"
date: 2026-04-02T06:16:32+00:00
draft: false
slug: trinity-large-thinking
tags: [open-weights, reasoning, moe, inference]
categories: [models]
params:
  author: AI Beat Desk
  summary: >-
    Arcee AI released Trinity Large Thinking on April 1 — the reasoning-optimized
    variant of their 400B sparse MoE, trained by a 30-person startup on 2,048 Nvidia
    B300 GPUs. It ranks #2 on PinchBench for agentic tasks at roughly
    96% lower cost than the top model, under Apache 2.0. The architecture — 256
    experts with 4 active per token — is worth understanding.
---

[Arcee AI](https://www.arcee.ai/blog/trinity-large-thinking) shipped Trinity Large Thinking on April 1, the reasoning-optimized checkpoint in their Trinity family. The release is worth more than a quick look for a few reasons, starting with the fact that a 30-person company trained a 400B-parameter model from scratch and is giving it away under Apache 2.0.

## The architecture

Trinity Large uses sparse Mixture-of-Experts with 256 experts per layer, of which 4 are active at any given token. Total parameter count is ~398B; active compute per token is ~13B. The base model was trained on 17 trillion tokens across web, code, math, and reasoning data, running on 2,048 Nvidia B300 GPUs — which Arcee calls the largest publicly stated pretraining run on B300 machines.

The sparsity ratio here is extreme. Most MoE deployments run something like top-8 of 64 experts; Trinity's 4-of-256 means roughly 1.5% of experts fire per token. The upside is inference throughput: Arcee claims 2–3x faster generation than comparable dense models. The downside is that routing at this level of granularity is tricky to get right — you need strong load balancing or a handful of popular experts eat all the traffic while most sit idle. The team used momentum-based expert load balancing and z-loss regularization to keep routing stable during training.

Context support is 512K tokens, with a pretraining window of 8K extended during mid-training.

## Preview versus Thinking

The earlier Trinity Large Preview was a lightly post-trained instruct model — useful, but noticeably weaker on multi-step agent tasks. The Thinking variant adds extended chain-of-thought post-training with a stronger focus on multi-turn tool calling and long-horizon coherence. Arcee says it's "far better than Preview at multi-turn tool use, context coherence, and instruction following across long-horizon agent runs."

On [PinchBench](https://openrouter.ai/arcee-ai/trinity-large-thinking) — Kilo's benchmark for agentic model capability — Trinity Large Thinking ranks #2, just behind Claude Opus 4.6. The cost differential is roughly 96%: Thinking runs at $0.90/M output tokens versus Opus 4.6's pricing. For agentic workloads where token counts pile up across long sessions, that gap matters in practice.

## Why the license matters

Apache 2.0 is important here. The Trinity base model was trained by a team that includes DatologyAI and Prime Intellect as collaborators, and the weights are on [HuggingFace](https://huggingface.co/arcee-ai/Trinity-Large-Preview) in several quantizations (FP8, W4A16, GGUF). Permissive licensing means you can fine-tune it, run it behind a commercial API, or use it as a base for domain-specific post-training without navigating the custom license terms that trip up otherwise capable open models.

For comparison: the Trinity Large Preview had already served 3.4 trillion tokens on OpenRouter by the time Thinking dropped, making it the most-used open-weight model in the U.S. by that metric. The Thinking variant is likely to accelerate that, since agentic use cases are where token volumes grow fastest.

## What to take seriously here

A 30-person startup training a 400B model from scratch is not a normal thing. The capital required — 2,048 B300 GPUs, reportedly costing around $20 million — is real. The [technical report](https://arxiv.org/abs/2602.17004) is substantial. And the benchmark position is legitimate: #2 on an agentic benchmark with a credible methodology, not a cherry-picked academic task.

What's less clear is the Thinking variant's full benchmark profile. Arcee hasn't published a complete results table for it specifically; the numbers available are PinchBench and qualitative comparisons. MMLU, GPQA-Diamond, and math reasoning numbers for the Preview checkpoint are out there (GPQA-Diamond 63.3, AIME 2025 24.0), but the Thinking variant's reasoning lift on formal math benchmarks hasn't been published in detail yet.

Still: a fully open, commercially usable, 400B reasoning model at $0.90/M output tokens is a meaningful data point. The gap between what you can do with closed-API models and what you can do with open weights keeps narrowing.
