---
title: "ggml Ships 1.0.0"
date: 2026-11-14T06:17:26+00:00
draft: false
slug: ggml-one-zero
categories: [tools]
tags: [ggml, llama.cpp, local-inference, open-source, infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    After years of rapid iteration under opaque build numbers, ggml — the tensor
    library at the core of llama.cpp, whisper.cpp, and stable-diffusion.cpp —
    shipped its first 1.0.0 release on November 13. The milestone removes
    deprecated APIs, bumps the RPC protocol to v9, and delivers broad backend
    improvements spanning CUDA FP8, Metal MoE fusion, Vulkan FlashAttention, and
    new kernels for Hexagon, OpenCL, and WebGPU.
---

There's a certain kind of infrastructure project that everyone depends on but nobody quite treats as production-ready: it ships constantly, the API shifts under you, and you track it by commit hash rather than version number. [ggml](https://github.com/ggml-org/ggml) has been that project for local AI inference since Georgi Gerganov published it in 2023. Yesterday it tagged [v1.0.0](https://github.com/ggml-org/ggml/releases/tag/v1.0.0).

The practical scope of ggml is easy to understate. It's the tensor computation library that [llama.cpp](https://github.com/ggml-org/llama.cpp) is built on, which means it's the execution layer for most on-device language model inference outside of Apple's MLX and NVIDIA's TensorRT-LLM. It also powers whisper.cpp, stable-diffusion.cpp, and a long tail of other C/C++ inference runtimes. When ggml changes its API, all of them need to update.

## What 1.0.0 actually means

The release notes say it plainly: "ggml 1.0.0 prunes the deprecated public API." The long-deprecated `DIAG_MASK_INF` and `DIAG_MASK_ZERO` ops are gone, and the RPC protocol is bumped to v9 — peers on older versions will fail handshake rather than silently misbehave. Downstream code that relied on `ggml_diag_mask_inf` needs updating; there's an example fix for it in the commit list.

New surface area includes `ggml_rope_set_truncate` and `ggml_rope_get_truncate` for controlling YaRN correction-bound rounding behavior, and a backward pass for `TRI`. Neither of these is exciting on its own, but together the release says: we've defined what the stable API is, removed what it isn't, and set a contract downstream projects can version against.

## Backend work

The bigger substance is in the backends, which got substantial improvements across the board:

- **CUDA**: few-row quantized matmuls, OCP FP8 support, fused NVFP4 gate/up kernels
- **Metal**: fused MoE routing, which matters for the growing number of mixture-of-experts models running on Apple Silicon
- **Vulkan**: an Intel FlashAttention prefill kernel — Vulkan now has reasonable attention performance on Intel iGPUs
- **SYCL**: a device-side MoE scheduler
- **OpenCL**: decomposed FlashAttention for Adreno GPUs, which opens up better performance on Qualcomm mobile SoCs
- **WebGPU**: group norm, mean, and other new kernels
- **Hexagon**: expanded op coverage including Q5_0/Q5_1, Qualcomm's DSP accelerator becoming a more complete inference target

The scheduler also gained a small efficiency fix: it skips uploading inputs that aren't read by any kernel, reducing unnecessary data movement. MoE cache misses can now fall back to the CPU, which helps on systems where GPU memory is the binding constraint.

## Why this matters

The jump from v0.29.0 to v1.0.0 without an intermediate 0.30 is intentional — the versioning policy discussion that the release links to has been part of [ggml's roadmap](https://github.com/ggml-org/ggml) for a while. The 1.0 label isn't a claim that the library is feature-complete; it's a claim that the public API is now stable enough to promise you notice when it changes.

That's worth something. Local inference has gone from hobbyist experiment to production deployment fast enough that the ecosystem hasn't had time to formalize its dependency contracts. llama.cpp ships build numbers (`b10816`, `b10817`, ...) rather than semantic versions. Libraries that sit on top of ggml have been chasing a moving target. A versioned ggml makes it easier to write downstream code that breaks loudly rather than silently when the library moves.

The backend breadth is also a signal about where local inference is going. This is not a CUDA-only or Apple-only story anymore. Hexagon support, Adreno FlashAttention, Vulkan on Intel — ggml is tracking the full range of hardware where someone, somewhere, is trying to run a model.
