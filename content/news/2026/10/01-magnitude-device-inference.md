---
title: "Compile Once for Your Hardware"
date: 2026-10-01T08:30:00+02:00
draft: false
slug: magnitude-device-inference
categories: [inference]
tags: [inference, local-ai, performance, open-source, agents]
params:
  author: AI Beat Desk
  summary: >-
    Magnitude is a new open-source local inference engine that compiles
    device-specific kernels on first run rather than shipping precompiled
    binaries for broad hardware classes. The approach gets it to 92% faster
    decode on Apple Metal vs. llama.cpp and positions it specifically for
    agent workloads where decode throughput matters most.
---

There's a persistent friction in local inference: the kernels that ship with most runtimes are compiled for a hardware class — "Apple Silicon," "NVIDIA GPU" — rather than for the specific chip on your machine. The difference between an M4 Pro and an M3 Ultra matters for memory bandwidth, cache layout, and register count. Precompiled kernels leave that on the table.

[Magnitude](https://github.com/magnitudedev/magnitude), a YC S25 project that launched today, takes the opposite approach. On first run it performs device-specific compilation, tuning kernels against the exact hardware configuration it finds. Everything stays local — prompts, files, models — and subsequent runs use the compiled artifacts.

The performance claims are specific enough to be interesting: 92% faster decode on Apple Metal, 19% faster on CUDA, 2× overall versus llama.cpp. Decode throughput is the number that matters most for interactive and agent workloads. Prefill speed is usually fast enough that it's not the bottleneck; what you notice is how quickly tokens generate once the model starts producing output. A 92% improvement in decode throughput on Metal, if it holds across model sizes and use patterns, is the difference between a local model feeling sluggish and feeling usable for an agentic loop.

The agent framing is explicit in how they describe the tool — it's positioned not just as "a fast way to run models" but as an inference backend for existing agent frameworks: Pi, OpenCode, Hermes. The idea being that agent tasks are latency-sensitive in a different way than chat: the model may call a tool, wait for a result, and then need to reason about it, dozens of times in a single task. Shaving time from each decode step compounds.

The runtime supports Apple Silicon, NVIDIA, AMD, and CPU, releases under Apache 2.0, and is available on macOS, Linux, and Windows. The source is on GitHub and the license is permissive, which matters for embedding in other tools.

Whether the performance numbers generalize is the obvious open question. llama.cpp is a well-optimized baseline with years of hardware-specific tuning, and "2× overall" depends heavily on what hardware, what model size, and what workload. The Metal claim is the most plausible: Apple's GPU/CPU memory architecture has a lot of room for tuning that generic cross-platform runtimes don't fully exploit. The CUDA number (19%) is more modest and that's also the platform where llama.cpp has the most competition.

The device-specific compilation approach has a real cost: first run is slow while kernels compile, and the compiled artifacts are not portable. For a server deployment that's usually fine. For a desktop tool, it means users need to wait once, and the experience on first launch matters. The tradeoff is reasonable; it's the same tradeoff that XLA and TVM make in other contexts.

The broader picture is that local inference is getting serious engineering attention. Recent work has compressed 27B models to fit in under 6 GB, and edge inference tools have pushed smaller models onto microcontrollers. What Magnitude is doing sits in a different layer: not compression, not quantization, but runtime performance optimization for the hardware you actually have. That's complementary to the compression work happening elsewhere, and the combination means local inference is moving faster than the raw model size numbers suggest.
