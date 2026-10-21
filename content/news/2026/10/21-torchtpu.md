---
title: "TorchTPU: PyTorch on TPUs Without the Friction"
date: 2026-10-21T06:25:00+00:00
draft: false
slug: torchtpu-pytorch-tpus
categories: [infrastructure]
tags: [google, tpu, pytorch, inference, hardware, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Google has open-sourced TorchTPU, its long-planned successor to PyTorch/XLA,
    which brings native tensor integration and MPMD execution support to TPUs.
    The Fused Eager mode delivers 50–100%+ speedups over strict eager, and vLLM
    integration is on the roadmap — a meaningful step toward hardware-portable
    inference.
---

If you've ever tried to run PyTorch code on Google TPUs, you know the friction. [PyTorch/XLA](https://github.com/pytorch/xla) works, but it works on its own terms. Tensors are XLA-backed subclasses, not regular PyTorch tensors. Execution is purely SPMD — every rank runs the same program, synchronized. Debugging a shape mismatch or a trace error means reading through XLA's stack rather than PyTorch's. It's not unusable, but it demands a mental context-switch that makes TPUs feel like a different framework.

Google published a detailed technical post on TorchTPU back in April, outlining the design and promising an open-source release. That release arrived this week: the [public GitHub repository](https://github.com/google-pytorch/torch_tpu) is now live under Apache 2.0.

## What changes architecturally

The core difference from torch_xla is how tensors are represented. TorchTPU uses PyTorch's `PrivateUse1` interface — a dispatch key designed for exactly this kind of backend integration. Tensors on TPUs are regular `torch.Tensor` objects, not wrapped subclasses. Code that checks `isinstance(t, torch.Tensor)` or passes tensors to third-party libraries that don't know about XLA no longer needs to be handled specially.

The second change is MPMD support. PyTorch/XLA required SPMD: all ranks execute identical code, with partitioning expressed through annotations. TorchTPU supports per-rank divergent execution (multi-program multiple-data), which is how most distributed PyTorch code is actually written. Whether you're running pipeline parallelism, conditional logic on specific ranks, or just a single-device training loop, the model fits naturally.

## Three execution modes

The framework offers three eager modes:

- **Debug Eager**: Operations dispatch to CPU or standard XLA one at a time. Slow, but produces clean error messages.
- **Strict Eager**: Operations run on TPU without fusion. Correct, but slower than necessary because kernel launches are not amortized.
- **Fused Eager**: Operations are fused on the fly before dispatch. Google reports 50% to 100%+ speedups over Strict Eager.

For compilation, `torch.compile` with Torch Dynamo captures the FX graph, then routes through XLA as the backend (instead of Inductor), lowering to StableHLO. Custom kernels written in Pallas or JAX are also supported.

## Why this matters for inference

The release notes vLLM as a target integration. Right now, vLLM is CUDA-first — running it on TPUs requires patches and workarounds. If TorchTPU's native tensor representation and MPMD support land properly in vLLM, you'd get the same serving stack on TPUs without a separate codebase.

This is the piece that matters strategically. Training workloads are already somewhat hardware-diversified — JAX runs well on TPUs, and teams have tooling for it. Inference workloads are still heavily CUDA-dependent because the tooling (vLLM, SGLang, TensorRT-LLM) was built for it. If the inference stack becomes hardware-portable, TPUs become a credible alternative for serving, not just training.

How quickly that happens depends on how deeply vLLM and SGLang integrate TorchTPU versus treating it as an external compatibility layer. Google has the incentive; the implementation work is real; the result isn't certain. But the public repo means the surface area exists for that work to happen in the open now, rather than inside Google's deployment infra.

The project ships with example implementations for Llama, Qwen, ResNet, DLRM, minGPT, and UNet — breadth that suggests this is designed to be usable by the wider ecosystem, not just GooglePlex workloads.
