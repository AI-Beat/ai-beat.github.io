---
title: "AMD's FP8 Problem, and What It Costs"
date: 2026-06-03T06:16:00+00:00
draft: false
slug: deepseek-v4-flash-amd-mi300x
categories: [infrastructure]
tags: [infrastructure, amd, nvidia, inference, deepseek, fp8, hardware]
params:
  author: AI Beat Desk
  summary: >-
    A detailed engineering account of bringing DeepSeek-V4-Flash up on AMD
    MI300X reveals the real cost of AMD's software ecosystem gaps: FP8 format
    fragmentation, missing kernels, and HIP graph constraints that each
    required dedicated engineering effort before getting to 2,700 tokens/s.
---

The economics of the MI300X look compelling on paper. 192 GB of HBM3 versus the H100's 80 GB. Comparable FP8 throughput. Roughly half the list price. Currently available while H100 and H200 remain constrained. For running large inference loads — particularly large models with massive KV caches — the memory advantage alone is hard to ignore.

Then you try to actually run something on it.

[Doubleword's writeup](https://fergusfinn.com/blog/deepseek-v4-flash-mi300x/) on bringing DeepSeek-V4-Flash up on MI300X hardware is the kind of engineering account that tells you what the economics paper glosses over. Three distinct categories of problems, each requiring real work to fix, before the hardware's advantages became accessible at all.

## The FP8 Dialect Problem

The first issue is fundamental and affects anyone running a recent FP8-quantized model on older AMD hardware. AMD's MI300X uses an FP8 format called `fnuz` — Float 8, Negative Zero Unsigned. Newer AMD chips and the current version of vLLM use the OCP-standard FP8 format. These are not compatible. They produce different numerical results, and the difference isn't small: it's an off-by-factor-of-two error that makes quantized model outputs wrong in a way that isn't immediately obvious as a hardware bug.

The fix requires finding every place where quantization and KV cache operations make assumptions about the FP8 format and patching them for the MI300X variant. That's tractable but not trivial — quantization is threaded through a lot of inference code, and the failures are numerical rather than hard crashes, which means testing requires comparing outputs against a reference rather than just checking that nothing segfaults.

## Missing Kernels

The second problem is coverage. AMD's AITER kernel library — their answer to NVIDIA's cuBLAS, cuSPARSE, and the optimized attention kernels in FlashAttention — doesn't cover everything. Specifically, it lacks complete support for the gfx942 architecture (the MI300X) across the full set of operations that DeepSeek-V4's sparse attention requires: KV compression, learned block indexers, sliding-window handling.

The engineering path here was to route operations through AITER where kernels exist and fall back to Triton implementations otherwise. Triton is portable and correct, but it doesn't close the gap with hand-tuned NVIDIA kernels on NVIDIA hardware. Some operations run slower than they would on H100 not because MI300X is slower in principle but because the optimized implementation doesn't exist yet.

## HIP Graph Constraints

The third problem is architectural. HIP Graphs — AMD's equivalent of CUDA Graphs — record sequences of GPU operations for replay, which is how you amortize kernel launch overhead across many inference steps. The catch: recording requires pure device-side functions with static shapes. DeepSeek-V4's sparse attention is dynamic: metadata about which blocks to attend to changes per step, and correctly routing that metadata requires host-to-device scalar writes during graph capture.

The workaround involves restructuring the attention code to separate the static from the dynamic parts — keeping the heavy matmul operations inside the graph and handling the dynamic routing metadata outside it. This works, but it requires understanding the graph semantics well enough to split the operation cleanly, and it's not the kind of fix you find by reading documentation.

## What You Get After

After resolving all three categories of issues, Doubleword reports approximately 2,700 output tokens per second per GPU — a number they note comes with a caveat: "a meaningful slice of the time is not in the matmuls themselves but in the bookkeeping and tuning around them." That's an honest characterization of what mature vs. immature software ecosystems look like: even when the hardware is capable, software overhead eats into the numbers in ways that are hard to profile and harder to eliminate.

2,700 tokens/s is competitive. It's enough to make MI300X viable for production inference workloads. And the memory advantage is real — 192 GB means you can run models that simply don't fit in an H100's 80 GB, or run them with larger batch sizes and KV caches.

The point isn't that MI300X is bad. The point is that the gap between "theoretically competitive hardware" and "hardware you can actually ship on" is measured in engineering weeks, and that gap is entirely in the software layer. NVIDIA's moat isn't primarily the chips — it's the ten years of kernel libraries, profiling tools, and solved problems that mean you can run a new model on new hardware and have it mostly work the first time. AMD is closing that gap, but "mostly works after significant effort" is a different product than "mostly works immediately," and the difference has a real cost.

For teams with the engineering capacity to absorb that cost — and with workloads that genuinely need 192 GB HBM — MI300X is worth the investment. For everyone else, the economics paper still holds; the hardware is just more of a project than a drop-in.
