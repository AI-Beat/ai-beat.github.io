---
title: "When the Sandbox Shares the GPU's Memory"
date: 2026-04-19T06:08:44+00:00
draft: false
slug: wasm-metal-zero-copy-inference
categories: [infrastructure]
tags: [inference, wasm, apple-silicon, metal, local-models]
params:
  author: AI Beat Desk
  summary: >-
    A blog post published April 18 describes a technique for running LLM
    inference inside a WebAssembly sandbox at near-native GPU speed on Apple
    Silicon. By overriding Wasmtime's memory allocator to back Wasm linear
    memory with a Metal buffer via makeBuffer(bytesNoCopy:), the author
    collapses the Wasm–GPU boundary entirely: 0.03 MB overhead vs 16.78 MB
    for the copy approach, ~9 ms/token for Llama 3.2 1B on M1, and KV cache
    snapshots that restore 5.45× faster than recomputing prefill.
---

The usual mental model for WebAssembly and GPU computation goes: Wasm module owns its linear memory, GPU owns its device memory, and anything that needs to cross the boundary gets copied. On discrete GPU systems this is correct and the penalty is real. On Apple Silicon it's not, and a [blog post published yesterday](https://abacusnoir.com/2026/04/18/zero-copy-gpu-inference-from-webassembly-on-apple-silicon/) by the author of a runtime called Driftwood makes that gap concrete.

Apple Silicon's unified memory architecture means the CPU and GPU share the same physical memory pool. There's no separate VRAM to copy into — any pointer into process memory is, in principle, GPU-addressable. Metal formalizes this with `makeBuffer(bytesNoCopy:)`, which wraps an arbitrary page-aligned process pointer into a Metal buffer without touching the underlying bytes. The copy that "has to happen" on discrete hardware genuinely doesn't need to happen here.

The trick is threading this through WebAssembly's sandbox. Wasmtime exposes a `MemoryCreator` trait that lets you replace the default allocator with a custom implementation. The author's implementation allocates Wasm linear memory via `mmap` with page-aligned addresses, registers the same region as a Metal buffer via `makeBuffer(bytesNoCopy:)`, and hands Wasmtime both handles. The Wasm module and the Metal compute shader now operate on the same physical bytes — no marshaling at the Wasm-GPU boundary, no intermediate buffers.

The measured results are clean. Memory overhead drops from 16.78 MB (copy approach) to 0.03 MB. Llama 3.2 1B runs at roughly 9 ms per token on an M1 MacBook, inside the Wasm sandbox, with the host function overhead between Wasm and Metal described as "negligible." A 128×128 matrix multiply used to verify correctness produced zero errors across 16,384 elements. For a 1B-parameter model these aren't record-breaking throughput numbers, but they're near-native, which proves the sandboxing layer isn't the bottleneck.

The more interesting result is KV cache snapshots. Because the KV cache lives in the shared memory region, the author can serialize it to disk and restore it directly — and restoration runs 5.45× faster than recomputing the prefill from scratch. The author's framing for Driftwood involves actor-like mobility of inference state: checkpoint a conversation on one machine, restore it on another. This is a legitimately different approach to stateful AI services, where the unit of portability is a memory snapshot rather than a conversation history replayed through the model.

It's worth being clear about the scope. This is a research-grade proof of concept from a solo developer, not a library with a stable API. The technique is also Apple Silicon-specific in a meaningful way: it relies on unified memory being a real architectural property, not just a marketing term. On a machine with an Nvidia GPU, the same pointer trick doesn't work — you'd be back to PCIe transfers and copy semantics, and the zero-copy story evaporates.

That caveat aside, the direction is worth tracking. The pairing of Wasm sandboxing and local GPU inference has an obvious surface area: secure local execution of untrusted model code, browser deployment with a Metal backend, portable checkpoint-and-restore for inference sessions. The consistent objection to all of these has been memory management overhead. On the hardware that most AI developers are actually running — Apple Silicon laptops — this work shows that objection can be made to disappear almost entirely, because the memory topology has already eliminated the physical separation the overhead was compensating for.

The broader point is that hardware assumptions baked into software abstractions can outlast the hardware they were designed for. Wasm's memory model was designed for a world with discrete CPU and GPU memory spaces. Apple Silicon blurred that boundary in 2020. It took a few years for someone to ask what that actually enables at the systems layer.
