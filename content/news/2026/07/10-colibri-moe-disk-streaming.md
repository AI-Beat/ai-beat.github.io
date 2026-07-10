---
title: "Streaming 744 Billion Parameters from Disk"
date: 2026-07-10T06:12:16+00:00
draft: false
slug: colibri-moe-disk-streaming
categories: [inference]
tags: [inference, open-source, moe, consumer-hardware]
params:
  author: AI Beat Desk
  summary: >-
    Colibri, a ~1300-line pure-C engine posted on Hacker News overnight, runs
    the 744B GLM-5.2 MoE on a 25GB-RAM consumer machine by streaming routed
    experts from NVMe on demand. It's not fast, but it works — and the
    architectural insight it exploits (most of a MoE's parameters are cold at
    any given token) points to a design pattern that will matter more as
    open-weight frontier models keep growing.
---

A Hacker News [Show HN post](https://github.com/JustVugg/colibri) from yesterday got 524 upvotes for a simple reason: it does something that feels like it shouldn't work. [Colibri](https://github.com/JustVugg/colibri) runs GLM-5.2 — a 744-billion-parameter Mixture-of-Experts model — on a consumer machine with 25 GB of RAM. The whole thing is a single C file, roughly 1300 lines, with no external dependencies.

The trick is not compression or distillation. It's MoE sparsity, exploited more aggressively than most inference engines bother to.

## Why MoE architecture changes the math

Dense models are what they sound like: every parameter participates in every forward pass. A 70B dense model needs ~35–40 GB of memory just to hold its weights at float16. You can quantize down, but there's a floor; eventually you're degrading the model rather than just reducing precision.

MoE models work differently. GLM-5.2 has 744B total parameters, but only around 40B are activated per token. The architecture splits most of the feed-forward layers into "experts" — small sub-networks — and routes each token to just a handful of them via a learned gating function. The other experts sit idle. They have to exist (they define the model's capacity and what it learned), but they don't participate in any given inference step.

Colibri makes this literal. It separates GLM-5.2's weights by access pattern:

**Resident in RAM**: The dense components — attention layers, shared experts, embeddings — account for about 17B parameters. At int4 quantization, this fits in roughly 9.9 GB. These run every forward pass, so they stay in memory.

**Streamed from disk**: The 21,504 routed experts (75 MoE layers × 256 experts each, plus a multi-token prediction head) live on NVMe. Each expert is about 19 MB at int4, for a total of ~370 GB on disk. When a token's gating function selects eight experts from a layer, those eight are fetched from disk into a per-layer LRU cache.

There's also an optional "hot-store" — a pinned set of frequently accessed experts that stay in RAM — and the OS page cache acts as a free second level of caching on top of that. Asynchronous readahead prefetches likely-next experts while current computation runs.

## How fast is it?

Honest answer: not very. On the development machine (WSL2, 12 cores, 25 GB RAM, NVMe via VHDX), cold decode runs at 0.05–0.1 tokens per second. With the multi-token prediction head enabled, each forward pass produces 2.2–2.8 tokens, which helps throughput somewhat but not dramatically.

Community benchmarks on better hardware are more encouraging: a Ryzen AI 9 with optimized caching reached 0.37 tok/s, and an M5 Max hit 1.06 tok/s. The primary constraint is disk bandwidth — specifically, how quickly you can load ~11 GB of routed expert weights per token from NVMe into RAM. At 1 tok/s on the M5 Max, that's 11 GB/s of random reads, which maxes out even fast consumer NVMe.

Some HN commenters questioned whether 0.1 tok/s is usable for anything real. They're not wrong that it's impractical for interactive use. But the project makes a different kind of argument: that a 744B frontier-class model can answer correctly on hardware that costs a few hundred dollars, rather than requiring an H100 or a cloud bill. A pre-built int4 weight package for Colibri is on Hugging Face.

## The design pattern underneath

What Colibri is actually demonstrating is that MoE inference doesn't have to be treated as "one big blob that fits in memory or doesn't." The sparsity of expert activation is a structural property you can exploit at the system level: keep what's always hot in RAM, stream what's conditionally needed, cache aggressively at every level.

This isn't the first project to try disk-backed inference — llama.cpp has had mmap support for dense models for years, and running Llama weights off a slow drive is a known if painful option. But MoE models have a distinct advantage here: the token-level routing means only a small, predictable-ish subset of experts gets accessed per forward pass. A dense 744B model would need to read most of its weights for every token, making disk streaming untenable. GLM-5.2's sparsity makes Colibri's architecture coherent.

As open-weight frontier models continue their arms race in total parameter count — GLM-5.2's 744B won't be a ceiling for long — the question of how to run them without data-center hardware becomes more pressing. Colibri is a proof-of-concept that MoE's access patterns are exploitable enough to make consumer inference viable, even if you need patience measured in seconds per token.
