---
title: "One Model, One Chip, No Framework"
date: 2026-05-08T06:04:37+00:00
draft: false
slug: ds4-deepseek-metal
categories: [tooling]
tags: [inference, local-llm, deepseek, apple-silicon, quantization, moe]
params:
  author: AI Beat Desk
  summary: >-
    Salvatore Sanfilippo (antirez, Redis) released ds4: a single-model Metal
    inference engine for DeepSeek V4 Flash that deliberately rejects the
    general-framework approach. Asymmetric 2-bit quantization on MoE experts
    only gets a 280B-parameter model into 128 GB RAM with 26–36 t/s generation,
    1M-token context, and disk-persisted KV cache on Apple Silicon.
---

If you know antirez from Redis, you know the engineering aesthetic: do one thing, do it precisely, write tight C. Salvatore Sanfilippo applied the same instinct to LLM inference and released [ds4](https://github.com/antirez/ds4), a local inference engine for DeepSeek V4 Flash that works exclusively on Apple Silicon via Metal.

The README opens with what it isn't: "not a generic GGUF runner, not a wrapper around another runtime, and not a framework." It's a purpose-built C and Objective-C implementation for one model on one chip family, and that narrowness is the whole point.

## The Quantization Trick

Getting DeepSeek V4 Flash — a 284B-parameter sparse MoE model — into a 128 GB MacBook requires aggressive compression. The approach ds4 takes is asymmetric: only the routed MoE experts get quantized to 2 bits (up and gate projections at IQ2_XXS, down projections at Q2_K). Everything else — shared experts, the attention projections, routing weights — stays at full precision.

This is a deliberate bet on the structure of MoE models. The routed experts are the large, expendable components: for any given token, most of them are bypassed entirely, and their individual weights contribute less to output quality than the shared components that run every time. Compressing them aggressively while leaving the shared and routing infrastructure intact preserves the model's critical invariants. The reported quality delta from the asymmetric quantization versus full 4-bit is smaller than you'd expect for 2-bit, which validates the bet.

The practical outcome: a 280B model fits in memory that most people who can afford an M3 Ultra Mac already have.

## Context and Persistence

DS4 supports a 1-million-token context window, which on a local machine raises the obvious problem: the KV cache for a million tokens doesn't fit in memory. The solution is disk-based KV cache persistence. The cache pages to disk and survives between sessions, so you can continue a long conversation or a deep code review without recomputing the full context from scratch. This is a genuinely useful feature for extended work sessions that most inference frameworks treat as an afterthought.

Benchmarks on M3 Max (128 GB) show 58 t/s prefill on short prompts and 26 t/s generation — the latter is comfortably above the reading-speed threshold that makes a model feel interactive. For long inputs, the M3 Max prefill climbs to around 250 t/s; an M3 Ultra reaches closer to 468 t/s prefill and roughly 84 t/s generation. Community benchmarks in the HN thread report 20–50% faster throughput than llama.cpp on the same hardware and model, which aligns with what you'd expect from a runtime that has eliminated all generality overhead.

## The Meta-Layer

There's a small self-referential detail in the README: the project was "developed with strong assistance from GPT 5.5." It's worth noting — not as novelty but as signal. antirez writing production C has historically meant dense, hand-tuned code. The fact that he used a competing model to assist in writing an inference engine for a different competing model, and mentions it plainly in the docs, is a casual data point on how the tooling ecosystem has normalized.

The code is explicitly alpha. Metal-only means it doesn't run on Linux or Windows, and there's no CUDA path yet. The model weights format is custom (not standard GGUF), so you need the antirez-published GGUF files specifically. These are deliberate tradeoffs for the target audience: M3 Max or M3 Ultra owners who want a fast, stable local inference option for a capable MoE model.

DeepSeek V4 Flash (the model itself) was released in late April with a 1M-token context window and Apache 2.0 weights. DS4 makes it practically usable on high-end consumer hardware. Most people with a $5,000 Mac can now run a 280B-parameter model locally with 1M context, at speeds that don't require patience — which, a year ago, would have been a statement about data center hardware.
