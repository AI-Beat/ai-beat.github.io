---
title: "antirez Ports MiniMax H3 to Metal"
date: 2026-08-11T06:19:38+00:00
draft: false
slug: antirez-h3c-video-metal
categories: [inference]
tags: [inference, video, metal, apple-silicon, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Salvatore Sanfilippo — the author of Redis — published a native C+Metal inference
    engine for MiniMax H3 targeting M3/M5 Macs, roughly repeating what llama.cpp did
    for language models: bypass the Python stack, write tight hardware-specific kernels,
    and find out how fast the silicon can actually go.
---

Salvatore Sanfilippo — better known as antirez, the author of Redis — published [h3.c](https://github.com/antirez/h3.c) today: a native inference engine for MiniMax H3 written in C with Metal shaders, targeting M3 Max and M5 Max hardware. The project achieves around 74–77 seconds per end-to-end render at roughly 40 GB of peak unified memory, and is actively being optimized.

MiniMax H3 is the 33B open-weight omni-modal model that landed on Hugging Face in early August. It generates up to 15 seconds of 2K video with native stereo audio in a single forward pass and tops the open-weight video leaderboard — though its license [restricts use in the US, EU, UK, and South Korea](/news/2026/08/04-minimax-h3-open-weights-closed-markets/) over ongoing copyright litigation. That restriction applies to the model weights; h3.c itself is MIT-licensed and the code is usable regardless.

The llama.cpp parallel is obvious. When Georgi Gerganov wrote llama.cpp in early 2023, it wasn't that Python couldn't run LLaMA — it was that C with GGML kernels could make it fast enough to be genuinely useful on a MacBook. h3.c is the same bet applied to a video generation model: drop the PyTorch stack, write Metal kernels tuned to H3's actual compute patterns, and see how much performance the hardware can deliver. antirez notes the code contains contributions from liuliu (the developer behind DrawThings), suggesting some of the Metal work may find its way into that app if H3 support lands there.

The architectural difference from a standard transformer makes h3.c more interesting than a generic port. H3 uses a hybrid design that includes Mamba-style selective state-space model (SSM) blocks alongside conventional attention. SSM blocks have a fundamentally different memory access pattern from attention — linear in sequence length rather than quadratic, with per-timestep state updates instead of KV cache lookups. You can't drop in FlashAttention and call it done; the Metal kernels have to be written for H3's specific operation sequence. That's most of what the project is still working on: matching the optimized reference performance and then going further.

The practical situation: 40 GB peak memory means you need a Mac with at least 64 GB (and preferably 96 GB) of unified memory to run H3 without swapping, and render times are measured in minutes per clip even at the current state of optimization. That's broadly where llama.cpp stood in the first weeks of its existence — functional, slow, and getting faster as more people filed PRs. Whether h3.c reaches real-time or near-real-time throughput on high-end Apple Silicon within a few months is an open question, but the kernel infrastructure to get there is being built.

The thing antirez brings that matters most here isn't the Metal expertise — it's the instinct for optimization that comes from writing a widely-deployed database from scratch. Redis is built on a discipline of understanding what the hardware is actually doing at each instruction. That same lens applied to a video generation model's Metal kernels is worth watching.
