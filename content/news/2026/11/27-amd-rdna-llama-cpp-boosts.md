---
title: "What ROCm Left on the Table"
date: 2026-11-27T06:11:15+00:00
draft: false
slug: amd-rdna-llama-cpp-boosts
categories: [inference]
tags: [inference, amd, local, llama-cpp, open-source, hardware]
params:
  author: AI Beat Desk
  summary: >-
    A community project called llama-cpp-rdna-boosts has been building RDNA-specific
    patches onto a pinned llama.cpp commit — WMMA flash attention, adaptive MTP,
    fused MoE kernels, host-resident expert caching — and reporting roughly 2x prefill
    and 57% token-generation gains on RDNA4. The project also surfaces a worthwhile
    distinction: some inference forks improve benchmark numbers by changing which
    experts the model loads, not by improving hardware use.
---

Consumer AMD GPU users running local LLMs have always been in a worse position than their NVIDIA counterparts, and not primarily because AMD hardware is slower. It's because ROCm, AMD's compute platform, trails CUDA in both maturity and community tooling. The official ROCm/llama.cpp fork documents MI300X and other Instinct datacenter cards; consumer Radeon cards get mentions in troubleshooting threads. RDNA4 (the RX 9000 series) only hit the market in 2026, so there wasn't much time for the community to catch up anyway.

[llama-cpp-rdna-boosts](https://github.com/stew675/llama-cpp-rdna-boosts) is a project by GitHub user stew675 that has been filling that gap with a focused set of patches. Rather than maintaining a full fork, the approach is a small changeset applied to a pinned upstream commit, rebased periodically as llama.cpp moves forward. The last rebase was November 20. The project targets RDNA3 (`gfx1100`), RDNA3.5 (the Strix Point and Halo APUs at `gfx1150`/`gfx1151`), and RDNA4 (`gfx1200`/`gfx1201`), with best-effort RDNA2 coverage via community contributions.

The specific changes are worth reading through. For attention: BF16 KV cache, native-BF16 and WMMA flash attention, with per-arch head-size limits tuned by GPU family. For prefill: chunked gated-delta-net prefill using bf16/WMMA kernels. For decoding: adaptive MTP draft depth (accessible as `--spec-type draft-mtp-adaptive`), which can be combined with the `ngram-mod` draftless path. For MoE models: fused MoE and k-quant decode paths, plus a hybrid all-reduce on RDNA4 that falls back to RCCL elsewhere. There is also host-resident MoE expert caching and a streamed KV window that frees several GiB of VRAM at large batch sizes.

A [user post on devnull.land](https://devnull.land/amd-specific-llama-cpp) reported testing on an AMD R9700 in an eGPU setup with a Q6_K_L quant of Swift-1.5-Qwen3.8-27B, MTP enabled. Prefill rose from about 700 to about 1,300 tokens/second. Token generation went from about 35 to about 55 tokens/second. Those are meaningful numbers: 86% prefill improvement, 57% token-gen improvement, without changing the model or the quantization.

The post also makes a point worth passing on. The inference fork landscape has drifted toward a kind of benchmark inflation where "higher tokens/second" sometimes means the fork has changed *which* experts a mixture-of-experts model loads — preferring whichever experts happen to be in VRAM, rather than the ones the model's routing actually selects. That can produce larger benchmark numbers while silently changing model behavior. The claim about llama-cpp-rdna-boosts is that it isn't doing that: the gains are hardware-side, in kernel paths and memory management, not in expert selection policy.

That distinction matters if you care about the model's actual reasoning versus raw throughput on evals. Local inference has been getting crowded with projects that optimize for the number, and it's worth knowing whether a reported speedup is on the hardware side or the model side.

Access is straightforward: a prebuilt container image at `ghcr.io/stew675/llama-cpp-rdna-boosts:server-rocm-10.1`, runnable via podman with `/dev/kfd` and `/dev/dri` passed through. No manual ROCm install, no fighting with hipBLASLt. The project notes a known issue with ROCm nondeterminism (worked around with `ROCBLAS_USE_HIPBLASLT=0`) and recommends the TheRock ROCm runtime on Windows over the display driver's bundled copy.

If you have RDNA3 or newer hardware and you've been using CUDA-only benchmarks to decide whether to bother with AMD for inference, this is worth revisiting.
