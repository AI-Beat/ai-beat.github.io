---
title: "Eight Billion Active Parameters, Frontier Coding Results"
date: 2026-07-22T08:30:00+02:00
draft: false
slug: laguna-s2-open-weight-coding-moe
categories: [models]
tags: [open-source, coding, moe, poolside, agents, benchmark]
params:
  author: AI Beat Desk
  summary: >-
    Poolside released Laguna S 2.1, a 118B-total / 8B-active MoE coding model
    that scores 70.2% on Terminal-Bench 2.1 — above DeepSeek-V4-Pro-Max at 64.0%
    and Inkling at 63.8%, both of which are an order of magnitude larger by active
    parameters. The weights are open under OpenMDW-1.1.
---

Poolside [released Laguna S 2.1](https://poolside.ai/blog/introducing-laguna-s-2-1) yesterday, an open-weight coding model that makes a specific and verifiable claim: frontier-class agentic coding performance at mid-tier inference cost.

The numbers: 118 billion total parameters with 8 billion active per token. The architecture is a Mixture-of-Experts with 256 routed experts and a top-10 router plus one shared expert, 48 transformer layers in a 1:3 ratio of global to sliding-window attention, and a 1,048,576-token context window. Poolside trained it on 4,096 NVIDIA H200 GPUs starting May 22, finishing the run in under nine weeks.

On [Terminal-Bench 2.1](https://poolside.ai/blog/introducing-laguna-s-2-1) — a benchmark of long-horizon tasks evaluated through a live terminal connection — Laguna S 2.1 scores 60.4% without thinking mode and 70.2% with it. For comparison, DeepSeek-V4-Pro-Max, which has 1.6 trillion total parameters and roughly 49 billion active, scores 64.0%. Thinking Machines' Inkling (975 billion parameters) scores 63.8%. NVIDIA's Nemotron 3 Ultra (550 billion) scores 56.4%. On SWE-Bench Multilingual, Laguna S 2.1 reaches 78.5%.

The model runs on a single NVIDIA DGX Spark. At 8B active parameters, the inference math is similar to running an 8B dense model; you get the expressiveness of a much larger parameter space without paying for it at serving time.

What poolside built here is a domain-narrowing bet. They weren't trying to produce a competitive general model. Laguna was trained specifically for agentic coding: terminal interaction, long-context tool use, multi-step software tasks. The benchmark gap — 8B active beating 49B active by 6 points — is most easily explained by the specificity of that training objective aligning tightly with the specificity of the benchmark. Terminal-Bench 2.1 is exactly what Laguna was built for. Broader capability benchmarks would tell a more nuanced story.

That's not a knock against it. Specialization is a reasonable strategy when the deployment use case is well-defined, and "an agent that completes real coding tasks" is well-defined. The open-weight release under [OpenMDW-1.1](https://openmdw.org/license/) is useful: the license allows commercial use with some restrictions on fine-tuning for similar model releases, similar in character to Meta's Llama licenses. Poolside is also publishing all evaluation trajectories at trajectories.poolside.ai, which is a worthwhile gesture toward reproducibility given how easy it is to benchmark-overfit when your eval setup is opaque.

The model is available on Hugging Face and through OpenRouter and Vercel AI Gateway. The smaller Laguna XS 2.1 ships alongside it for cases where even 8B active is too much at inference time.

One thing worth watching: poolside has been building toward enterprise software development workflows, and a capable open-weight base model is useful both as a deployment target and as a signal of what the proprietary API-only version is doing. If the open model scores 70.2% on Terminal-Bench, it's a reasonable inference that the closed offering is meaningfully ahead of that.
