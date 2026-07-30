---
title: "31 Tokens Per Second from Disk"
date: 2026-07-30T08:12:12+02:00
draft: false
slug: turbo-fieldfare-gemma-4-apple
categories: [inference]
tags: [inference, apple, moe, open-source, gemma]
params:
  author: AI Beat Desk
  summary: >-
    TurboFieldfare, a Swift/Metal inference engine posted to Hacker News overnight,
    runs Gemma 4 26B in roughly 2 GB of RAM by keeping the model's shared core in
    memory and streaming routed experts from SSD via explicit pread I/O — achieving
    31–35 tok/s on an M5 MacBook Pro, well into interactive usability. It's the
    expert-streaming technique from Colibri, applied to a smaller MoE on purpose-built
    Apple hardware, and the throughput gap shows what platform-specific implementation
    buys.
---

Three weeks ago we covered [Colibri](https://github.com/JustVugg/colibri), a 1,300-line C engine that ran GLM-5.2's 744 billion parameters on a 25 GB machine by streaming expert weights from NVMe. The throughput was 0.05–0.37 tok/s — a proof of concept that the approach was possible, not that it was practical.

[TurboFieldfare](https://github.com/drumih/turbo-fieldfare), posted to Hacker News overnight with 720+ points, applies the same core idea to a different MoE on Apple Silicon and gets 31–35 tok/s on an M5 MacBook Pro. That's a qualitative shift. At 5 tok/s on an M2 Air it's usable; at 31–35 tok/s on M5 hardware it's faster than a comfortable reading pace.

## The model and the split

Gemma 4 26B-A4B has 26 billion total parameters but only roughly 3.88 billion active per token. Like all MoE architectures, it routes each token through a gating function that selects a small subset of "experts" — feed-forward sub-networks — from a much larger pool. The experts that don't participate in a given forward pass have to exist (they define what the model learned), but they don't have to be in RAM.

TurboFieldfare makes that split literal. The shared core — attention layers, shared experts, embeddings — compresses to about 1.35 GB in MLX 4-bit quantization (group size 64) with 8-bit routers. That stays resident. The routed experts live on disk, in a full installation of roughly 14.3 GB. At inference time, the routing function identifies which experts are needed for the current token, and TurboFieldfare fetches them from SSD.

The total RAM footprint is approximately 2 GB, including the FP16 KV cache for a 4K-token context. On a base 8 GB MacBook Air, that leaves 6 GB for the OS and whatever else you're running. A 26B model with room to spare.

## Why pread beats mmap

The implementation choice that makes the performance numbers work is explicit `pread` I/O rather than memory-mapped files. With `mmap`, the OS manages expert loading via page faults: readable in principle, but each fault adds latency the application can't control. TurboFieldfare's creator measured ~10 ms per expert with `mmap` and ~2.8 ms with direct `pread`. That roughly 4× difference per expert load compounds across the hundreds of expert fetches that happen during a generation sequence.

On top of `pread`, each transformer layer gets a 16-slot LFU (least-frequently-used) cache. According to the creator's measurements in the HN thread, approximately 41% of experts selected for one token are reused for the next — natural language has enough local regularity that the routing function tends to pick similar experts for consecutive tokens. Cache hits skip disk entirely.

Chunked prefill processes the input prompt in groups of up to 128 tokens, so a single expert fetch serves multiple prompt positions simultaneously. This reduces time-to-first-token without requiring the full expert set to be loaded at once.

## What the throughput gap means

The contrast with Colibri isn't primarily about the model size difference (744B vs 26B), though that matters. It's about what platform-specific implementation unlocks. Colibri is pure C and portable; its bottleneck is the raw disk-to-RAM bandwidth available on commodity hardware. TurboFieldfare is Swift and Metal, targeting Apple Silicon specifically. M5's SSD bandwidth — needed to sustain the stream of expert reads — is substantially higher than what most x86 consumer machines offer, and Metal's compute pipeline reduces the overhead between fetches.

The 48 tok/s one community tester reported on an M4 Max with 64 GB RAM is a useful ceiling: when RAM is large enough that the OS page cache retains experts across the first generation pass, subsequent passes see few disk reads at all. At that point, "streaming from disk" becomes "streaming from warm OS cache," and throughput climbs past what the SSD alone can sustain.

Whether this generalizes up to larger MoE models depends on how the math scales. Gemma 4 26B's 3.88B active parameters means each token requires fetching a small fraction of the total expert pool. A model with the same MoE fraction but 4× the total parameters would need 4× the disk reads, at which point even M5's SSD bandwidth may become the bottleneck. But for a 26B MoE in 2026, on hardware that costs under $2,000, the numbers are good enough that "download and run locally" is a real recommendation rather than a caveat.
