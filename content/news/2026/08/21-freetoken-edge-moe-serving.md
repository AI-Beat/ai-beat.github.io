---
title: "The 753B Model on Your Workstation"
date: 2026-08-21T06:20:00+00:00
draft: false
slug: freetoken-edge-moe-serving
categories: [inference]
tags: [inference, edge, moe, local-models, open-source, hardware]
params:
  author: AI Beat Desk
  summary: >-
    FreeToken, from a team at UC Berkeley and MIT, proposes a serving stack
    that treats a personal machine's CPU, GPU, and RAM as a single elastic
    compute surface, adapting to what's actually available rather than
    committing to a fixed offloading strategy. The result: a 35B model on
    an 8GB laptop GPU, 284B on a gaming desktop, and the 753B GLM-5.2
    on a single workstation with one high-end GPU.
---

Running large language models locally has always been a game of creative memory accounting. llama.cpp popularized CPU offloading; exllama pushed the GPU envelope; various tools have tried hybrid approaches. What all of them share is that you configure a strategy upfront and stick to it: a certain number of layers on GPU, the rest on CPU, and some tolerance for the memory pressure when those boundaries collide.

[FreeToken](https://arxiv.org/abs/2608.16157), from researchers at UC Berkeley and MIT, takes a different position. Personal hardware is heterogeneous and time-varying — the GPU and CPU contend with other processes, available bandwidth changes as you switch tasks, and the balance between fast GPU memory and slower CPU RAM differs dramatically between a gaming laptop and a workstation. Rather than asking you to configure a static allocation, FreeToken continuously maps computation and model state onto whatever is actually available.

The paper calls this bandwidth-adaptive execution. The serving runtime monitors resource availability and rebalances accordingly, keeping the inference pipeline full rather than waiting for stalled memory transfers. MoE models make this tractable in a way that dense models don't: at any given token, only a small fraction of experts activate (roughly 2-3% for most current architectures), which means the working set is smaller and more dynamic than the total parameter count implies. The router knows which experts will activate before the compute starts, so FreeToken can speculatively load them while earlier layers are still running.

The headline numbers are striking. An 8GB laptop GPU serves a 35B parameter model at 39 tokens per second. A 32GB gaming desktop runs 284B interactively. A single-GPU workstation handles the 753B GLM-5.2 at double the throughput of competing systems. Tail latency is kept below 44 seconds TTFT across all workloads; the baselines all exceed 150 seconds. On smaller models like Qwen3.6-35B-A3B, the system achieves 1.5–2.3× higher decode throughput than current SOTA edge serving.

The agent angle is worth noting separately. Most local inference benchmarks test static generation: throughput, latency, perplexity. Agent workloads look different — repeated short generations with tool calls, context growth across many turns, and patterns that change as the agent moves through different task phases. The evaluation runs four scenarios directly: AIME math reasoning, a SWE-bench coding agent via OpenCode, a concurrent multi-agent setup using the Claude Code protocol, and an email/calendar agent with 13 turns per session. FreeToken includes explicit handling for agentic state reuse: KV cache from prior turns is held in a tiered memory structure, so the overhead of re-processing context on each call is reduced. It's a practical optimization that matters a lot for the actual use case but rarely appears in benchmark comparisons.

The author list is notable: Shuo Yang, Matei Zaharia, Song Han, Ion Stoica, Kurt Keutzer, and others from Berkeley and MIT — the same group that has produced much of the foundational work in distributed LLM inference. FreeToken isn't a side project.

There's a relevant question the paper doesn't fully answer: what does throughput look like compared to naive CPU offloading at the same hardware? The bandwidth-adaptive approach should be better in theory, but the margin matters for whether this is a meaningful improvement in practice or an elegant system with modest gains over simpler alternatives. The paper focuses on what's possible rather than how much faster it is than alternatives.

Regardless, the capability threshold here is real. If you can run a 753B model on a single workstation — not a cloud instance, not a server rack — the calculus around local deployment shifts. Models at that scale now compete with hosted API endpoints on capability grounds. Privacy-sensitive workloads and offline environments get access to frontier-class open weights. The gap between "a model you can run" and "a model worth running" has been narrowing; FreeToken pushes it closer.

The paper is at [arXiv:2608.16157](https://arxiv.org/abs/2608.16157). Code and binaries are available at [flashml.ai](https://flashml.ai), with source at [GitHub](https://github.com/FlashML-org/FreeToken).
