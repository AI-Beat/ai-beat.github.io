---
title: "Teaching a 4B Model to Outplan Postgres"
date: 2026-09-17T06:09:17+00:00
draft: false
slug: postgres-query-planner-rl
categories: [rl-training]
tags: [rl-training, databases, fine-tuning, grpo]
params:
  author: AI Beat Desk
  summary: >-
    Rohan Bansal trained a 4B parameter model to generate Postgres join-order hints
    via distillation from a frontier model followed by reinforcement learning with
    a custom anchored GRPO variant — yielding a 1.81× geometric mean speedup on
    113 join-heavy queries for about $1,200 in total compute. The methodology shows
    how cheap specialist RL is getting for niche performance optimization tasks.
---

Postgres's query planner, when faced with a complex multi-table join, uses dynamic programming up to a certain join count and then falls back to heuristics. For queries with many tables, the search space is too large to explore exhaustively, so the planner makes educated guesses. Those guesses are usually good. "Usually good" leaves room for a model.

That's the premise of [a project Rohan Bansal published yesterday](https://rohanbansal.com/qorl): fine-tune a 4B parameter model to generate planner hints — annotations that steer Postgres toward a specific join order — and see if it can beat the default optimizer. On 113 join-heavy queries from the Join Order Benchmark, the trained model achieves a 1.81× geometric mean speedup, with overall latency down 44.7%.

The approach uses Postgres's hint mechanism (pg_hint_plan). The model receives a query and emits a set of join operator and order hints; if a hint is invalid, Postgres ignores it and falls back to its own plan. This matters: the floor is stock Postgres performance, not a crash. A model that's uncertain can produce a bad hint, lose nothing, and let the planner do its job.

Training happened in two stages. First, off-policy distillation: Bansal used GPT-6 Astra to generate successful hint trajectories on the benchmark, then trained the 4B model to imitate those. This gave the smaller model the vocabulary — what a valid hint looks like, how the agent harness is structured — without requiring it to bootstrap the task from scratch. After distillation, the model could produce valid plans for 14 of 113 queries.

Then comes RL. The second stage used a variant of GRPO — Group Relative Policy Optimization — that Bansal calls "anchored." The challenge is measurement noise: query execution times fluctuate based on buffer cache state, OS scheduling, concurrent load. A naive reward signal (was this plan faster than baseline?) produces noisy gradients and unstable training. The anchored approach pairs candidate and default measurements during inference, interleaved, with the shared buffer pool fixed at 2GB to force consistent page cache behavior. It's the kind of domain-specific engineering that makes the difference between a training run that converges and one that doesn't. After RL, the model succeeds on 101 of 113 queries.

The cost angle is what makes this worth paying attention to beyond the benchmark: total project spend was roughly $1,200, split between a consumer GPU rig for Postgres measurements and a rented 2×H100 node for training and inference. Database query optimization is a mature, deeply engineered domain — commercial databases have decades of heuristics baked in — and a small model is genuinely competitive with niche RL training, done on a budget that a startup or individual engineer can afford.

The natural extensions are obvious: other databases, online learning from production traces, extending from join order to index selection or parallel worker count. The Join Order Benchmark is a controlled workload, not production traffic with shifting cardinality and query patterns. But the methodology is sound. Careful reward design, measurement discipline, and two-stage training (distillation then RL) form a reusable recipe. As inference gets cheaper and small-model RL becomes more routine, training a per-application optimizer policy starts to look less like a research project and more like a systems engineering technique you'd reach for when the query planner keeps making bad calls on a specific workload.
