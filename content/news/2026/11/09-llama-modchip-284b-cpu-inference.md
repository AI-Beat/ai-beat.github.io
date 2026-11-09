---
title: "284 Billion Parameters, 64 Gigabytes of RAM"
date: 2026-11-09T06:18:34+00:00
draft: false
slug: llama-modchip-284b-cpu-inference
categories: [inference]
tags: [quantization, open-source, local-inference, deepseek, cpu]
params:
  author: AI Beat Desk
  summary: >-
    Synchopate's llama-modchip runs DeepSeek-V4-Flash (284B MoE) on a CPU-only
    64GB DDR5 machine at 9–14.7 tok/s — no GPU required. The engineering is
    real but the memory margin is razor-thin, the format is non-standard, and
    the interesting question isn't whether it fits but what you'd actually use
    it for.
---

The claim from [llama-modchip](https://github.com/synchopate/llama-modchip) is blunt: take DeepSeek-V4-Flash, a 284.33-billion-parameter mixture-of-experts model, and run it on a consumer CPU with 64GB of DDR5 and no GPU. The compressed model file lands at 56.56 GiB; peak resident memory at 256K context is 58.9 GiB. On a machine with exactly 64GB total, that leaves about 5 GiB for the operating system and everything else running alongside it.

The test hardware was an AMD Ryzen 9 9955HX — Zen 5, 16 cores — which is a laptop chip, not a workstation. That part is worth sitting with for a moment: a laptop-class processor is now the floor, not the ceiling, for running a 284B model from scratch.

## How it fits

Several techniques combine to get the model under 64GB. The main compression pass is **Fractal-64**, a custom quantization format with a custom tensor type (ID 40) that isn't compatible with standard GGUF tooling. That alone doesn't account for the full size reduction — the project also applies **SwiGLU-H2 compression**, a second-order Hessian-based method that exploits the activation structure of the SwiGLU gating used in DeepSeek's MLP blocks. On top of that, **L2-quadrature MoE pruning** trims the expert routing paths, which matters a lot for a model where most parameters live in the expert layers.

The runtime side targets the memory hierarchy directly. Weights are tiled across ZMM AVX-512BW registers and L1D/L2/L3 cache using a four-level scheme, designed to minimize DRAM bandwidth pressure during token generation. On DDR5, bandwidth is the actual bottleneck for CPU inference at this size — you're streaming tens of gigabytes per generated token at 2–3 tokens/sec without this, so the cache tiling is doing real work.

## Speculative decoding

Decode performance without speculative decoding is 9.06–10.74 tok/s on the Zen 5 machine. With the project's **Stockfish-PVS** speculative decoder — which uses a 1.64MB draft component called RouteCrack — that rises to 11.5–14.7 tok/s. The draft acceptance rate on Nemotron-3.5-30B-A3B is 96.4%, which is unusually high and suggests RouteCrack is well-calibrated to the expert routing patterns of these MoE architectures. Prefill (prompt processing) runs at 43–44 tok/s.

For comparison: the original DeepSeek-V4-Flash in GGUF format would need roughly 110GB of RAM to run with standard llama.cpp quantizations, which puts it on a dual-socket server or a Mac Studio with 192GB unified memory. llama-modchip is claiming to do the same job at 64GB — roughly half the memory — through aggressive compression.

## The friction

The Fractal-64 format is not standard. These model files won't load in mainline llama.cpp, Ollama, or LM Studio without patching. That's a significant ecosystem barrier: the entire local inference toolchain you've built around GGUF doesn't work here. You're adopting a custom runtime from a project that is, as of this writing, a one-person GitHub repository with a permissive Apache-2.0 license but no guarantee of continued maintenance.

The memory margin is also genuinely tight. 58.9 GiB peak in a 64GB machine assumes almost nothing else is competing for RAM. Run a browser tab or two in the background and you're paging. The project's own documentation notes the dependency on AVX-512 — without it, the cache tiling approach falls back to slower paths, and the numbers don't hold.

## What it means

The "run a 284B model locally" angle grabs attention, but the more interesting question is for whom this is actually useful. Interactive chat at 10 tok/s is workable but not fast. Batch processing — summarization pipelines, offline document analysis, evaluation runs — is where CPU inference at this scale starts to look genuinely attractive, because latency matters less and you're not competing for GPU slots.

The practical ceiling is probably machines with more RAM. A 192GB unified-memory Mac Studio runs this with 130GB to spare; a dual-socket Threadripper workstation with 256GB ECC can run two instances. The llama-modchip numbers for Nemotron-3.5-30B on the Threadripper are telling: up to 111.6 tok/s. CPU inference, properly optimized, isn't just a curiosity at the smaller model scales.

The standard framing for local inference has been about VRAM tiers: 24GB gets you a 70B, 80GB gets you a 180B, and so on. llama-modchip is making an argument that the VRAM ceiling is avoidable if you're willing to do the compression engineering. Whether that argument generalizes — whether the quality loss from 3.75 bpw plus MoE pruning is acceptable for your use case — is something you'd have to test against your own workload. The project publishes the numbers but not independent quality evals. For a 284B MoE at this compression, some degradation is likely, but the degree matters enormously.

The code is on GitHub under Apache-2.0 at [github.com/synchopate/llama-modchip](https://github.com/synchopate/llama-modchip), currently at v2.4.0.
