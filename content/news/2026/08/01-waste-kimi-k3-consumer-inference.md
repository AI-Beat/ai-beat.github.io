---
title: "One pread Per Expert"
date: 2026-08-01T08:15:00+02:00
draft: false
slug: waste-kimi-k3-consumer-inference
categories: [inference]
tags: [inference, open-source, kimi-k3, moe]
params:
  author: AI Beat Desk
  summary: >-
    SQLiteAI released WASTE, a dependency-free C inference engine that runs
    Kimi K3's 2.78-trillion-parameter model on 29GB of RAM at 0.50 tok/s by
    keeping the resident trunk in memory and streaming activated experts from
    NVMe with a single pread() per expert.
---

Kimi K3 is a 2.78-trillion-parameter model. The weights occupy 1.42TB in MXFP4 format. Running it at full throughput requires 18+ H100 80GB GPUs. [SQLiteAI's WASTE project](https://github.com/sqliteai/waste) runs it on 29GB of RAM at 0.50 tokens per second.

That number — half a token per second — is the first thing to sit with. It's not fast. A two-hundred-word response takes about eight minutes. Most applications would find this unusable. But the fact that it's possible at all, with a self-contained C library and no external dependencies, is a concrete demonstration of what mixture-of-experts sparsity actually means for hardware-constrained inference.

The key insight is that K3 only activates roughly 50 billion of its 2.8 trillion parameters per token — under 2%. The model has a 16-of-896 expert routing scheme: of the 896 experts in each MoE layer, only 16 fire for any given token. The remaining 880 contribute nothing to that computation. WASTE (Weight-Aware Streaming Tensor Engine) exploits this directly: keep the ~27GB resident trunk — the parts of the model that are always active — in RAM, and stream just the activated experts from NVMe as they're needed.

The design that makes this practical is the cost model. Routing to an expert costs exactly one `pread()` — a single unbuffered disk read. Expert weights are stored in residual vector quantization at 3 bits per weight, packed into 4KB-aligned records that match filesystem block boundaries. One expert, one syscall, one aligned read. No seeks between the trunk loading and the expert data, no scattered I/O patterns.

The remaining RAM after the trunk becomes an expert cache governed by an LFRU (Least Frequently Recently Used) policy. WASTE automatically calculates the cache budget by stepping down in whole multiples of one token's working set, which for K3 is roughly 17GB. So on a machine with 64GB RAM, you'd get about two tokens' worth of cached experts beyond the trunk — not a lot, but enough to see repeated activation patterns benefit from warm cache.

The KV cache for attention layers gets a separate optimization: the paper absorbs the projection matrices into the cache representation, which reduces cache overhead by 53× compared to standard implementations. This matters because KV cache is what grows with context length — for long conversations, even at 0.5 tok/s, you'd otherwise blow the memory budget.

SQLiteAI makes [sqlite-vec](https://github.com/asg017/sqlite-vec) and related embedded data tools — single-file, dependency-free, C-native. WASTE fits exactly in that ethos. No CUDA, no BLAS, no Python runtime. You're meant to be able to ship this as a library and have it run anywhere. The Apache 2.0 license is in keeping with that.

There's an interesting parallel to [TurboFieldfare](https://github.com/drumih/turbo-fieldfare), which appeared here last week — that's a Swift/Metal engine that runs Gemma 4 26B on Apple Silicon Macs by streaming experts from SSD. Both projects are exploiting the same fundamental property of MoE models: activated sparsity makes SSD-to-RAM streaming viable in a way that dense models would never permit. For a dense 1.4T model you'd need every weight for every token; the streaming approach would fall apart immediately.

The difference is scale and speed. TurboFieldfare gets 5-35 tok/s on M-series hardware with a 26B model that fits in a reasonable memory footprint. WASTE gets 0.5 tok/s with a 2.78T model that doesn't. TurboFieldfare is approaching usable; WASTE is demonstrating a lower bound.

The meaningful takeaway isn't "you can run K3 on a gaming PC today." It's that the combination of MoE architectures and fast NVMe storage creates a hardware access gradient that didn't exist with dense models. As NVMe read speeds increase and expert caching gets smarter, the tok/s ceiling on these consumer-hardware approaches goes up without any changes to the model weights themselves. WASTE at 0.5 tok/s today might be 5 tok/s in two years with the same code on better hardware — not because the model changed, but because the I/O bandwidth did.

That's a different kind of consumer inference story than quantization. It's not about making the model smaller; it's about matching the model's own sparsity to the hardware's actual I/O profile.
