---
title: "Kimi K3.1 Pro: Scaling RL Across Domains"
date: 2026-10-24T06:11:38+00:00
draft: false
slug: kimi-k3-1-pro-rl-hallucination
categories: [models]
tags: [kimi, moonshot, reinforcement-learning, open-weight, hallucination]
params:
  author: AI Beat Desk
  summary: >-
    Moonshot AI's Kimi K3.1 Pro is a post-training update to Kimi K3 that cuts
    claim-level hallucination by two-thirds via joint RL across hundreds of
    domains simultaneously. The RL scaling story is technically interesting; the
    open weights arrive November 4.
---

Moonshot AI [announced Kimi K3.1 Pro](https://www.kimi.ai/blog/kimi-k3-1-pro) yesterday as a post-trained update to Kimi K3, the 2.8-trillion-parameter open-weight model the company released in July. The headline improvement is a sharp reduction in hallucination: on Moonshot's internal Kimi Hallu Bench Hard (a stress test, not everyday conditions), claim-level hallucination fell from 23.4% to 8.3%. That's a roughly two-thirds reduction from the same base model, achieved purely through post-training rather than architecture changes.

The mechanism is what Moonshot calls "scalable RL" — joint reinforcement learning across hundreds of domains simultaneously, rather than fine-tuning for specific tasks. The idea is that narrow post-training produces models that improve on the target distribution while regressing elsewhere; running RL jointly across many domains at once forces the optimization to find improvements that generalize. Whether that framing holds up to scrutiny would require a proper technical report, which hasn't been released yet. But the benchmark pattern is consistent with the claim: K3.1 Pro improves across coding, finance, factual accuracy, and long-horizon reasoning without obvious tradeoffs appearing in the published numbers.

The most interesting vignette in the announcement is a walking simulator example: the model was given a reinforcement learning task to train a simulated human to walk, and in open-loop probes of the simulator's dynamics, it found 36 errors in shoulder-muscle moment-arm calculations before retraining the controller. That's a qualitative description of something that looks like debugging a physics simulation, not just optimizing a reward signal — the kind of behavior that suggests the model is doing something structurally more interesting than "predict next token." It's a cherry-picked example, and we should read it skeptically, but it's the kind of thing that makes the RL story plausible rather than purely marketing.

K3.1 Pro also gets a 2–3× faster sibling — K3.1 Pro Highspeed — available through the same API. The API model IDs are `kimi-k3.1-pro` and `kimi-k3.1-pro-highspeed`, with zero data retention available for API customers. Full model weights are due November 4.

One thing worth noting is the safety tradeoff Moonshot chose to surface directly: they removed cyber-related data from the mid-training and RL stages. On xdev-bench-hard, this model scores 81.0% compared to 90.5% for the same pipeline without those removals, and it remains below frontier proprietary models on offensive security benchmarks. That's an unusually transparent disclosure for a model release — publishing the cost of the safety decision in percentage points rather than just asserting the model is safe.

The benchmark comparisons Moonshot chose to publish are all against GPT-6 Astra and Claude Opus 5.5. On Terminal-Bench 4.0, K3.1 Pro scores 65.7% against GPT-6 Astra's 58.2% and Claude Opus 5.5's 66.4%; on BenchCAD with tools it scores 92.7% against 95.9% and 96.2% respectively. Those are competitive numbers at the 3T parameter scale, and the open-weight release arriving November 4 means anyone can run the model rather than just read the benchmarks.

The broader pattern from K3 to K3.1 is that post-training is doing real work at this scale: hallucination down, multi-domain performance up, and a faster variant for latency-sensitive applications — all from the same base weights. That's a more tractable engineering story than "train a bigger model," and it probably generalizes beyond Moonshot.
