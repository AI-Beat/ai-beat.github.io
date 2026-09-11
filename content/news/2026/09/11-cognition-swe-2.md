---
title: "Coding on the Pareto Frontier"
date: 2026-09-11T06:08:15+00:00
draft: false
slug: cognition-swe-2
categories: [models]
tags: [models, coding, reinforcement-learning, efficiency]
params:
  author: AI Beat Desk
  summary: >-
    Cognition released SWE-2, a coding model post-trained on Kimi K3 2.8T using
    cost-penalized reinforcement learning. It scores 50% on FrontierCode 1.1
    at 64% lower cost than Fable 5.1 and takes 58% fewer turns than its
    predecessor — the result of directly optimizing for the cost/quality tradeoff
    rather than treating efficiency as an afterthought.
---

[Cognition released SWE-2](https://cognition.com/blog/swe-2) yesterday, a coding agent model built by post-training [Kimi K3](https://arxiv.org/abs/2505.15000), Moonshot AI's 2.8-trillion-parameter open model, using what they call Pareto-optimized reinforcement learning. The headline numbers — 50% on FrontierCode 1.1, competitive with Fable 5.1 at 64% lower cost, beating its own predecessor SWE-1.7 on DeepSWE 1.1 by 35 percentage points (37.7% to 73.0%) — are solid. But the more interesting part is the training approach.

## The cost-penalized reward formula

Most coding agent evaluations ask one question: did it solve the task? Cognition's insight for SWE-2 was to fold cost directly into the reward signal during training. Their formula is \(R = S - \lambda_e C\), where \(S\) is a success score, \(C\) is cost (measured in tokens used across the agent's trajectory), and \(\lambda_e\) is a per-effort-level coefficient tuned to match the Pareto frontier slope at that operating point.

The effect is that the model learns to solve problems in fewer turns, not just to solve them. On FrontierCode tasks, SWE-2 medium scores higher than SWE-1.7 while using 58% fewer turns and costing 81% less on average. That's not a compression artifact — the model is genuinely doing less unnecessary work, not just truncating its output. The verification discipline shows up too: Cognition reports the model runs more targeted tests, writes shorter but higher-coverage test suites, and abandons dead ends faster.

Training all effort levels simultaneously matters here. The coefficient \(\lambda_e\) varies per effort tier, so the model learns distinct behaviors for cheap-and-fast vs. expensive-and-thorough modes rather than learning one behavior and trying to threshold it post hoc. This is a fairly direct application of multi-objective RL — the Pareto front is explicit in the training objective rather than being a byproduct of deployment tuning.

## Standing on Kimi K3

Cognition didn't train the base model. Kimi K3 had already undergone substantial RL for coding tasks before they got it, which means SWE-2's post-training was working with a strong prior rather than from a generic instruction-tuned checkpoint. Whether this gives Cognition a structural advantage or just a faster start depends on how much of the behavior is locked into the base model vs. shaped by post-training — something they don't address directly.

The benchmarks Cognition chose deserve a close look. [FrontierCode 1.1 Main](https://cognition.com/blog/frontiercode-1-1-release) is their own benchmark, measuring whether AI-generated pull requests would realistically be merged by a human maintainer. [DeepSWE 1.1](https://www.cognition.ai/blog/deep-swe-bench) is also Cognition-adjacent. Self-selected benchmarks aren't automatically suspect — building a good benchmark is real work and labs often use their own tools — but readers should weight external benchmark results (Terminal-Bench 2.1 at 92.8%, Terminal-Bench 4 at 27.3%) alongside the headline numbers.

Terminal-Bench 4 at 27.3% is worth noting specifically. It's the hardest version of that benchmark and the model leaves roughly three quarters of it unsolved. That's not a criticism — FrontierCode and Terminal-Bench 4 are measuring different things, and no current model has a strong Terminal-Bench 4 score — but it makes clear that the efficiency gains are concentrated in the task distribution SWE-2 was optimized for.

## What it gets right

The explicit cost modeling in the training objective is the genuinely novel piece here. Inference efficiency is usually treated as a systems problem: quantize the weights, optimize the attention kernel, run speculative decoding. SWE-2 treats efficiency as a behavioral property that should be trained directly, which is a different framing. If the behavior generalizes well — if a model that learned to be efficient on coding tasks is also efficient on other agentic tasks — that has implications beyond the coding domain.

The model is available in Devin Desktop and Devin CLI at launch; no standalone API or weights are published. For teams already in the Devin ecosystem this matters immediately. For everyone else it's a technical story about a training approach that, if it generalizes, will show up in other models too.
