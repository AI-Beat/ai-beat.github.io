---
title: "Streaming 104 GB of MoE from Disk"
date: 2026-09-02T08:00:00+02:00
draft: false
slug: slotstream-moe-ssd-streaming
categories: [inference]
tags: [inference, moe, apple-silicon, open-source, mlx]
params:
  author: AI Beat Desk
  summary: >-
    Slotstream is a Swift/MLX tool that runs Qwen3.8-Flash-Next — a 125B MoE
    weighing 104 GB at 4-bit — on Apple Silicon Macs with far less RAM than
    the model weighs, by streaming routed expert weights on demand from NVMe
    SSD into a fixed cache-slot pool.
---

There is a design property of mixture-of-experts models that is easy to overlook because most deployment infrastructure ignores it: at any given forward pass, only a small fraction of experts activate. For Qwen3.8-Flash-Next, a 125B MoE with 68 GB of routed expert weights spread across 48 layers, the vast majority of that 68 GB is idle at any given moment. The dense trunk — the attention layers, embeddings, norms — is only 3.8 GB.

[Slotstream](https://github.com/carloslfu/slotstream) is a new Swift/MLX tool that exploits this directly. The dense trunk lives in RAM; the routed experts live on NVMe and get loaded with `pread` into a fixed pool of cache slots shared across all 48 layers. When a forward pass needs an expert, it reads from disk into a slot; when the slot is needed again, it evicts to make room. The result is that a model requiring 104 GB can run on an Apple Silicon Mac with the minimum RAM floor planned at around 8.1 GB — provided the SSD has roughly 110 GB free.

The reason this is feasible at all on Apple Silicon is the NVMe bandwidth. M-series Macs top out around 7–7.5 GB/s sequential read. Compared to a GPU system that moves data from VRAM to compute in a couple hundred GB/s, that sounds painful — but MoE inference is compute-light per expert call, and the routing gate has already selected only the experts actually needed before any I/O happens. The sparsity is doing real work here. Whether throughput is usable depends heavily on how many experts fire per token and whether your SSD is anywhere near saturated by other activity, but the project reports real inference running on M3 hardware.

The implementation exposes an Ollama-compatible API and supports standard sampling knobs (temperature, top_p, top_k, min_p, presence_penalty, seed, stop sequences). Conversation follow-up turns only re-prefill the new tokens, so time-to-first-token stays roughly flat as a chat grows, which is important when every prefill pass is already touching disk.

There are obvious caveats. You are not getting datacenter inference speeds. The project itself says a Mac with 512 GB of storage is the "realistic minimum." SSD wear is a consideration for sustained use, though the access pattern is reads only. And this is an early-stage project — a Swift weekend hack, effectively — not a production serving stack.

But the framing matters. A frontier-class model running on a laptop, with no GPU and no cloud bill, is a genuinely different situation from "a quantized 7B that fits in unified memory." The 104 GB number is real model capacity, not a compromise. The engineering trick here — fixed-size disk-to-slot mapping, shared across layers, triggered by the router — is clean enough to generalize, and it would not be surprising to see something similar show up in llama.cpp or MLX upstream before long.

The repository is at [carloslfu/slotstream](https://github.com/carloslfu/slotstream).
