---
title: "Training the Thinking Short"
date: 2026-09-28T08:12:19+02:00
draft: false
slug: ember-1-reasoning-efficiency
categories: [models]
tags: [models, inference, reasoning, efficiency, fireworks]
params:
  author: AI Beat Desk
  summary: >-
    Fireworks Research released Ember-1, a post-trained variant of Kimi K3 that
    produces roughly 40% fewer reasoning tokens while holding quality steady across
    benchmarks and live A/B tests. The work points at a new cost axis in the
    reasoning model era: not smaller models, not quantization, but training the
    internal monologue to be shorter.
---

Reasoning models have a different cost structure than standard LLMs. Most of what you pay for isn't the answer — it's the scratchpad. A model thinking through a coding task might produce fifteen hundred tokens of internal reasoning to get to a fifty-token solution. That asymmetry is fine for accuracy, but it compounds quickly in production: a billing line that's 90% "thinking tokens" gets expensive fast, and latency goes with it.

[Fireworks Research released Ember-1](https://fireworks.ai/blog/ember-1) this week, built on Moonshot's open-weight Kimi K3. The goal wasn't to shrink the model or reduce its capability — it was to train it to reason more concisely. After more than 50 training experiments, they say they found a regime where the model learns to drop redundant reasoning steps while preserving the ones that actually matter. The result: 35–50% fewer reasoning tokens across seven benchmarks, with comparable answer quality.

The real-world validation is more persuasive than the benchmark numbers. Fireworks ran live A/B tests on two production customers' traffic and measured about 35% fewer tokens per task with no detectable quality degradation — internal developers running the model daily didn't notice a difference. That's a harder test than a leaderboard.

What makes this technically interesting is the axis. The usual options for cheaper inference are: use a smaller model (accept quality loss), quantize (modest gains, quality risk), or cache aggressively (helps with repeated prompts). Ember-1 targets something different: the actual reasoning behavior. The model isn't being compressed; it's being trained to think differently.

Kimi K3 is open-weight, which is what makes this possible. Fireworks built their own training service infrastructure, took the base weights, and post-trained specifically for reasoning efficiency. That's a straightforward pipeline if you have the compute, but it requires the weights to be available in the first place. Closed models don't offer this path — you can't post-train GPT-6.

There are open questions. Shorter reasoning traces might mask the quality loss rather than eliminate it: a benchmark measures whether the final answer is correct, not whether the reasoning path was robust enough to handle harder variants of the same problem. The 50-experiment count also suggests the optimization landscape is tricky — finding the right balance between concision and correctness isn't a one-shot tuning job. Fireworks is framing this as a research preview with a two-week window, noting the permanence depends on demand.

Still, the direction makes sense. Reasoning tokens became a new cost axis the moment extended chain-of-thought went mainstream. It was always plausible that models could be trained to be less verbose about it. Ember-1 is one of the first examples of that being done in production and measured against real traffic — worth watching even if the model itself gets superseded soon.
