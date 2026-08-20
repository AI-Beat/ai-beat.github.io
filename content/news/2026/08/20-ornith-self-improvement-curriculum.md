---
title: "Ornith-1.5 Teaches Itself to Code"
date: 2026-08-20T06:14:00+00:00
draft: false
slug: ornith-self-improvement-curriculum
categories: [research]
tags: [models, training, reinforcement-learning, open-source, coding]
params:
  author: AI Beat Desk
  summary: >-
    Ornith released a three-size open-weight model family trained entirely by
    its own curriculum — the model proposes tasks, builds evaluation scaffolds,
    and generates rollouts, with a multiplicative reward structure that prevents
    gaming any one signal. DeepSWE jumped from 8.0 to 56.0; Terminal-Bench
    reached 86.1. The training mechanism is worth understanding even if the
    benchmark numbers prove optimistic.
---

The phrase "self-improvement" in machine learning usually conjures something dramatic — a model rewriting its own weights at runtime, or some form of recursive self-enhancement spiraling toward some capability horizon. [Ornith-1.5](https://ornith.ai/ornith_1_5.html) is more mundane and, for that reason, more credible: it automates the design of its own training curriculum. The weights are still updated on an offline cluster in the normal way. What the model generates is the content of that training — the tasks, the evaluation environments, and the solution attempts.

The released family comes in [three sizes](https://ornith.ai/ornith_1_5.html): 9B dense (runs via Ollama on a laptop), 35B MoE (single RTX 4090), and 397B MoE. All are MIT-licensed.

## How the loop actually works

Training proceeds through three interconnected components that are jointly optimized with [GRPO](https://arxiv.org/abs/2402.03300).

First, a **task proposer** generates new coding tasks. Each proposed task is scored on three criteria: validity (does it form a verifiable environment?), frontier difficulty (does the current model succeed roughly 20% of the time?), and novelty (is it sufficiently distinct from previously generated tasks?). The 20% success target is a specific choice — it keeps tasks at the edge of current capability, where gradient signal is meaningful, without pushing into regimes where the model almost never succeeds.

Second, a **scaffold generator** builds the evaluation harness for each task: the tools, decomposition strategy, and orchestration used to approach it. The scaffold is scored on whether it's aligned with the task, faithfully reflects solution quality, and is resistant to reward hacking.

Third, **rollouts** — the actual solution attempts — are scored on task success.

The multiplicative reward structure, V × D × N for task proposer and similarly for the harness, is the part that matters most. If any component fails, the joint reward collapses to zero. The model cannot game frontier difficulty by picking easy tasks, because novelty and validity also have to pass. It cannot rig the evaluation environment, because harness fidelity is independently scored.

## The benchmarks

Ornith-1.5-397B scores 56.0 on [DeepSWE](https://deepswe.com/), up from 8.0 for Ornith-1.0. The 9B variant scores 47.0, reportedly outperforming much larger models. On [Terminal-Bench 2.1](https://terminal-bench.com/), the 397B model reaches 86.1 against Claude Code's 85.2.

Two things about these numbers. First, DeepSWE figures come from Ornith's own reporting, and a jump from 8.0 to 56.0 in one generation has not been independently replicated. That doesn't mean the improvement isn't real — the training method is principled and the jump is plausible given how much the curriculum changed — but the absolute score deserves skepticism until others run it. Second, the Terminal-Bench comparison is more trustworthy since it's a third-party benchmark, though methodology differences (which scaffold, which Claude Code version) can shift numbers by several points in either direction.

## Why the mechanism matters more than the benchmark

Whether Ornith-1.5-397B actually beats Claude Code on Terminal-Bench in six months won't matter much — both systems will have moved on. What's more durable is the training approach.

Open-weight model quality has historically been constrained by the availability of labeled, high-quality data for complex tasks. If a model can generate its own tasks and evaluation harnesses that remain faithful to real-world quality (the hard part — reward hacking is a genuine risk here, which the harness reward attempts to address), the ceiling shifts. The bottleneck moves from data curation to compute and RL optimization stability.

Ornith-1.0 introduced self-scaffolding in [June 2026](https://ornith.ai/ornith_1_0.html). The jump to 1.5 closes the loop: instead of fixed human-authored scaffolds, the model now proposes both tasks and their evaluation environments. The 397B MoE scale is large enough to generate reasonable curriculum quality; the 9B model, with a score of 47.0 on DeepSWE, suggests the approach isn't purely a large-model phenomenon.

The code and weights are available now. Independent benchmark runs will tell us more than Ornith's own numbers can.
