---
title: "The Expert That Lives on Disk"
date: 2026-08-04T06:14:02+00:00
draft: false
slug: swiftlet-moe-expert-streaming
categories: [inference]
tags: [inference, apple, moe, open-source, on-device]
params:
  author: AI Beat Desk
  summary: >-
    Swiftlet is a new Swift+Metal runtime that runs 80B-parameter Qwen3-Next
    in 4.3 GB of RAM on an M5 Mac — not through compression magic, but by
    exploiting a structural property of MoE models: they activate only about
    3B parameters per token regardless of total size. The missing piece is fast
    enough SSD I/O to stream expert weights on demand, which Apple silicon
    happens to provide.
---

The headline figure is 4.3 GB of peak RAM for an 80B-parameter model. That sounds wrong until you look at what's actually happening.

[Swiftlet](https://github.com/leonickson1/Swiftlet) is a Swift and Metal runtime for the Qwen3-Next and Qwen3.5/3.6 hybrid MoE families, published this week as a working end-to-end implementation. Its trick isn't compression — it's architecture. Qwen3's models are Mixture-of-Experts hybrids: at any given token, the model routes through a small number of active expert subnetworks while the rest sit idle. For the 80B variant, only about 3 billion parameters actually compute on each forward pass. The other 77 billion are bystanders.

Most inference runtimes ignore this and load everything into RAM anyway, because the alternative — fetching weights from disk mid-inference — sounds too slow to be practical. Swiftlet argues this assumption doesn't hold on Apple hardware. The dense core of the model (the non-expert layers) stays resident in memory, which keeps the fast path fast. The routed expert weights, which change token by token, are packed into `.qpack` containers using fixed-stride blobs: every expert is exactly one `pread` syscall from storage. Fetching isn't random-access into a multi-gigabyte mmap; it's a predictable, tight I/O pattern that Apple's NVMe controllers and unified memory architecture handle well.

The numbers bear this out. On an M5 Mac with the 35B Qwen3.6-A3B model (4-bit quantized), Swiftlet achieves 7–11 tokens per second using 2.6 GB of peak RAM with 18 GB on disk. The 80B Qwen3-Next-A3B gets 4.5–5 tok/s in 4.3 GB peak RAM. For reference, naively loading either model would require 18–42 GB of RAM and would be impossible on most Macs. The disk is doing the work that VRAM used to do.

The iPhone 17 result is the most surprising. The 35B runs at around 1 token per second using 2.5 GB of RAM on device. The author calls this the first time a model of this class has run natively on a phone, which is probably accurate. 1 tok/s isn't fast for chat, but it's fast enough for background tasks, and the model isn't quantized into unusability — it's a full 4-bit version of a 35B MoE hybrid.

One technical detail in the README stands out: the decode loop is currently dispatch-bound, not IO-bound. That means the SSD isn't the bottleneck — Metal kernel dispatch overhead is. The streaming approach has more headroom than the current numbers suggest, since the obvious optimization path (better batching, kernel fusion, reducing dispatch calls) doesn't require touching the storage layer at all.

The model also has an architectural property that happens to be useful here: it uses Gated DeltaNet linear attention layers rather than standard softmax attention, which means there's no KV cache growing over the context window. For a system where memory is the primary constraint, eliminating cache growth is a quiet advantage.

This isn't the first project to try weight streaming from disk for large models — llama.cpp has CPU-side offloading, and there are various mmap-based schemes. What's different here is the combination: a purpose-built Swift+Metal stack that runs natively on Apple silicon, a model architecture where sparsity makes streaming tractable, and SSD hardware that's fast enough to keep up. The approach doesn't generalize cleanly to dense models, where you'd need to stream most of the model for every token. But for MoE hybrids with aggressive routing sparsity, disk is genuinely viable as the weight store.

The project is marked as working end-to-end with validated output, and is actively under development on kernel performance. If the dispatch overhead gets addressed, the token rate on M5 would likely climb meaningfully before hitting storage as the new ceiling.
