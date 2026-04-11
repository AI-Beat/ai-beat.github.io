---
title: "Read First, Then Code"
date: 2026-04-10T06:22:29Z
draft: false
slug: research-driven-agents-skypilot
tags: [agents, coding, optimization, performance]
categories: [agents]
params:
  author: AI Beat Desk
  summary: >-
    SkyPilot published an experiment where giving Claude Code research papers
    to read before it optimized llama.cpp's CPU backend yielded 15% faster
    text generation on x86 for about $29. The interesting part isn't the
    speedup — it's that the literature revealed operator fusions that simply
    don't exist in source code, and a code-only agent had no way to find them.
---

The interesting part of [SkyPilot's research-driven agent experiment](https://blog.skypilot.co/research-driven-agents/) isn't the 15% speedup. It's where the speedup came from.

The setup: a Claude Code agent was tasked with optimizing the CPU inference backend of [llama.cpp](https://github.com/ggerganov/llama.cpp). For 3 hours across 4 cloud VMs (an Intel Xeon x86 instance and an ARM Graviton3), the agent ran over 30 experiments, writing benchmarks, testing candidates, and iterating. Total cost: roughly $29 — about $20 in compute and $9 in API calls.

The baseline run, where the agent went directly into the source code and started proposing optimizations, mostly went nowhere. The compute-bound improvements it found — targeting FLOP throughput — yielded negligible gains, around 1%. The reason, diagnosed only after the fact: the actual bottleneck was memory bandwidth, not compute. The code alone didn't make that clear.

The research-augmented run worked differently. Before touching the source, the agent read academic papers on CPU inference optimization and studied competing forks of llama.cpp — other implementations of the same model that had taken different optimization paths. That step changed what it looked for. Specifically, the literature pointed it toward operator fusions present in the CUDA and Metal backends but absent from the CPU backend: RMS norm combined with a subsequent multiply, softmax fused into the attention loop, and a three-pass QK attention tile reduced to a single FMA (fused multiply-add) loop with AVX2/NEON intrinsics.

Those fusions weren't findable by reading the CPU source code. They existed in a different part of the codebase (CUDA), in a different project (a competing fork), and in papers describing techniques that hadn't been ported to the CPU path. A code-only agent, even one reading the entire repository, had no path to discovering them.

The final numbers: text generation speed on x86 improved from 41.37 to 47.62 tokens/second (+15.1%). On ARM Graviton3, from 94.07 to 98.77 (+5%). Prompt processing was modest (around +1–2% on both architectures). Five out of thirty-plus experiments succeeded.

One way to read this: it's a demonstration that domain knowledge isn't always inside the codebase. For optimization work specifically, many good ideas start as papers, migrate to GPU implementations where they're immediately useful, and eventually make it to CPU backends — but that last step often lags by months or years. A model that only reads code misses the papers-to-implementation pipeline. One that reads the literature can spot the gap and fill it.

The SkyPilot team frames this as "research-driven agents," drawing an analogy to how a senior engineer approaches an unfamiliar codebase: you don't just grep and read; you learn the field first. That's a reasonable analogy, though it has limits — a senior engineer also builds intuitions from years of practice that a model is approximating from training data. The agent still needed those 30 experiments to find 5 that worked, which is a real inefficiency.

What's practically useful here: if you're running agents on optimization or performance work, seeding them with the right reading list may matter more than giving them a bigger codebase context. The $29 total cost for a 15% throughput improvement on an inference workload is obviously favorable — but the more durable takeaway is methodological. Research-first is a cheap input that changes what hypotheses the agent generates.

The full writeup on the [SkyPilot blog](https://blog.skypilot.co/research-driven-agents/) includes the benchmark setup, the specific kernel changes, and the cost breakdown.
