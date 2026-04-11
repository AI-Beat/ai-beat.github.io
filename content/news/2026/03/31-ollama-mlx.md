---
title: "Ollama Switches to MLX and Doubles Decode Speed"
date: 2026-03-31T06:29:50+00:00
draft: false
slug: ollama-mlx-apple-silicon
tags: [inference, apple-silicon, ollama]
categories: [infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    Ollama's preview MLX backend replaces direct Metal calls on Apple Silicon
    with Apple's dedicated ML framework, yielding a 93% decode speedup for
    Qwen3.5-35B-A3B on M5 chips. The update also adds NVFP4 quantization and
    a smarter KV cache — including prefix-aware eviction that keeps shared
    system prompts hot across conversations.
---

Ollama [shipped a preview](https://ollama.com/blog/mlx) of its MLX-powered backend for Apple Silicon on March 30, switching from direct Metal GPU calls to Apple's dedicated machine learning framework. On an M5 with more than 32GB of unified memory, the change is hard to dismiss: Qwen3.5-35B-A3B goes from 58 to 112 tokens per second at decode, and prefill climbs from 1,154 to 1,810 tokens per second.

The prefill number matters for RAG and long-context workloads, but for interactive use, decode is the bottleneck that actually feels slow. 112 tok/s for a 35B model means responses appear fast enough that the model stops feeling like it's thinking. At 58 tok/s, a 500-token response takes about 8.5 seconds. At 112 tok/s, the same response lands in under 4.5 seconds. The difference is visceral.

## Why MLX specifically?

MLX is Apple's answer to PyTorch and JAX for its own silicon — a NumPy-like framework that targets the unified memory architecture shared between CPU, GPU, and Apple Neural Engine. By going through MLX rather than Metal directly, Ollama gains access to the GPU Neural Accelerators introduced in M5, M5 Pro, and M5 Max. These are dedicated inference units that Apple hasn't fully exposed through Metal's lower-level API, so frameworks that go through MLX can use them while those hitting Metal directly cannot.

The practical implication is that the improvement is chip-generation-specific. M3 and M4 users running this preview will see some gains from better kernel implementations, but the sharpest numbers are for M5 hardware.

## NVFP4 and the production parity argument

The update also brings [NVFP4](https://ollama.com/blog/mlx) quantization support. This is NVIDIA's 4-bit floating-point format — the same format used in B200-class datacenter deployments — and Ollama frames its inclusion as enabling "production parity": the same model checkpoint that runs in a cloud inference farm can now run locally on a Mac.

That's a meaningful claim if you're developing an application against a locally-hosted model and want to validate that your prompts behave the same way they will in production. Quantization format mismatches between local and hosted models are a real source of evaluation drift, and NVFP4 support narrows that gap. Less clear is whether the quantization quality matches what NVIDIA's own kernels achieve on CUDA hardware — Apple's NVFP4 kernels are new and untested in the field.

## KV cache improvements

Three changes to KV cache management ship alongside the MLX backend. The most interesting is prefix-aware eviction: shared prefixes — system prompts, few-shot examples, long context preambles — now persist in the cache when shorter branches get evicted. This is significant for multi-turn chat with a fixed system prompt, and for RAG pipelines where the retrieved context often shares a long invariant prefix across queries.

The second improvement is cache reuse across conversations, which reduces peak memory consumption when you're running multiple sessions. The third is intelligent checkpoint placement that snapshots the cache at strategic points, reducing the prompt-processing overhead when a partially-filled cache is reused.

The 32GB requirement is real and meaningful — this restricts the preview to the higher-end Mac configurations. A fully loaded MacBook Pro M5 with 64GB or a Mac Studio M5 Max with 128GB will see the full benefit; a base-model MacBook Air will not.

The MLX backend remains in preview, which in practice means rough edges are expected and the default backend hasn't changed. Users need to opt in explicitly. Given that it's an open-source project, the preview will likely absorb community fixes quickly — but for production use in inference services, the stable release is worth waiting for.
