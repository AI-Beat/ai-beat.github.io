---
title: "DeepSeek Opens the Inference Stack"
date: 2026-10-21T06:20:00+00:00
draft: false
slug: deepseek-opens-inference-stack
categories: [inference]
tags: [deepseek, inference, open-source, kernels, hardware]
params:
  author: AI Beat Desk
  summary: >-
    DeepSeek has released DeepInfer, the inference framework behind its production
    services, connecting FlashMLA, DeepGEMM, DeepEP, and TileLang into a single
    stack under Apache 2.0. Cross-hardware support for NVIDIA SM100 and Huawei
    Ascend 950 from day one makes this more than just another vLLM fork.
---

For the past year, DeepSeek has been opening up its infrastructure in pieces. [FlashMLA](https://github.com/deepseek-ai/FlashMLA) dropped in February — optimized attention kernels for multi-head latent attention. [DeepGEMM](https://github.com/deepseek-ai/DeepGEMM) followed with fine-grained FP8 matrix multiplication. [DeepEP](https://github.com/deepseek-ai/DeepEP) covered the communication layer for MoE expert parallelism. Each release was useful, but fragmentary: you had the bricks without the mortar.

[DeepInfer](https://github.com/deepseek-ai/DeepInfer) is the mortar. It's the inference framework that DeepSeek uses to run V4.1-Flash in production, now released under Apache 2.0.

## What's inside

The design is built around the custom kernels DeepSeek has already shipped. FlashMLA handles the attention. DeepGEMM handles matrix multiplications. DeepEP handles the expert routing and communication. TileLang contributes additional fused operations. DeepInfer orchestrates all of these into a coherent serving pipeline with async scheduling, speculative decoding via draft tokens, and structured output (regex, JSON schema, or grammar).

One technically interesting piece is **bounded sliding-window attention replay**. Multi-turn agentic workloads often repeat context across requests. Rather than recomputing prefills in full, the system can replay a bounded window of prior computation — a form of speculative reuse that trades a small amount of correctness risk for meaningful prefill savings. Deterministic sampling is also supported, which matters if you care about reproducibility in production evals or CI.

The parallelism model covers three dimensions: attention data parallelism, context parallelism for long sequences, and MoE expert parallelism with load balancing. These mirror the decompositions DeepSeek described in their technical reports, so the framework structure is consistent with what they've described running at scale.

## Cross-hardware from day one

The most striking detail in the requirements table is the hardware list: **NVIDIA SM100 GPUs** and **Huawei Ascend 950 NPUs**. Both are explicitly supported from release.

SM100 is NVIDIA's Blackwell Ultra generation — high-end, not universal. This isn't a framework you can drop onto an A100 cluster and use as-is. DeepSeek appears to be targeting specifically the hardware they run in production and the hardware Chinese cloud providers commonly deploy.

The Ascend 950 support is less surprising in context — DeepSeek has clear strategic reasons to run on Huawei silicon given export restrictions — but it's rare to see explicit dual-hardware support in an inference framework at this level of optimization. Most frameworks treat non-CUDA hardware as a best-effort afterthought. DeepInfer lists Ascend 950 alongside SM100 in the requirements table, with separate install instructions for CANN 9.2.

## What's missing

DeepInfer covers *offline* inference only. There's no KV cache management system in this release, no prefill/decode disaggregation, and no load balancing across replicas. These are significant gaps for a production serving deployment — but they're also exactly the pieces that are hardest to generalize and easiest to build wrong in an open-source release. DeepSeek is clear that the framework was built primarily on vLLM, with design input from SGLang and TensorRT-LLM.

The honest reading is that this is a well-engineered foundation with useful novel pieces — the kernels, the parallelism model, the cross-hardware support — but it's not a drop-in replacement for a full serving stack. If you're running inference at DeepSeek-level scale, you'd need to bring the rest.

For researchers and engineers building on the stack, though, the value is real. The kernel integrations are production-validated, the cross-hardware path is documented, and Apache 2.0 means you can build on it without license friction. The missing pieces are the kind that different organizations want to implement differently anyway.
