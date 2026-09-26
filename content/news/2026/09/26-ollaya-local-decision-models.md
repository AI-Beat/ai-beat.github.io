---
title: "Jev on Your Own Metal"
date: 2026-09-26T06:12:04+00:00
draft: false
slug: ollaya-local-decision-models
categories: [tools]
tags: [tools, inference, open-source, jev, laya]
params:
  author: AI Beat Desk
  summary: >-
    Ollaya ships as a Rust daemon that pulls open decision models (Laya, Decider,
    NLI, GLiClass, Von, Qwen3guard) locally and serves them behind a TypeSafe-compatible
    API. Set an environment variable and existing Jev clients point at localhost —
    8–10 ms per five-question request on a 4090, no API key, Apache-2.0.
---

A week ago we wrote about [Laya and the Jev paradigm](../20-laya-before-jev/): the idea of routing structured questions through a small, fast decision model rather than spinning up a full LLM for every classification or routing task. The pitch was sound — single forward pass, calibrated probabilities, millisecond latency — but it required a TypeSafe API key and the usual tradeoffs that come with SaaS inference.

[Ollaya](https://ollaya.dev/) closes that loop. Released this week, it is a Rust daemon and CLI that pulls open decision models by name and serves them locally behind TypeSafe-compatible endpoints. The package description is accurate: it is Ollama for decision models.

## How it works

The CLI interface will feel familiar to anyone who has used Ollama: `ollaya pull laya`, `ollaya run laya`, `ollaya list`. The daemon starts on localhost and serves `/v1/systemone` and `/v1/models` with TypeSafe's request and response shapes. The official TypeSafe Python SDK works unchanged; switching from the hosted API to local inference is one environment variable.

Under the hood, Ollaya uses ONNX Runtime for inference — fp16 on NVIDIA GPU, fp32 on CPU — and ships model files as "small ONNX graphs, about 3 MB each" sourced from the original authors' Hugging Face repositories under Apache-2.0. The precision story is careful: the documentation describes verification that fp32 exports match PyTorch reference outputs on 2,383 questions per checkpoint, which is the kind of detail that matters if you're relying on these outputs for routing or guard decisions.

On an RTX 4090, five questions to Laya take about 8–10 ms end to end through the HTTP API. On CPU the latency goes up substantially, but for many classification workloads — log triage, real-time routing, spam detection — that is still orders of magnitude faster than LLM inference.

## The model roster

Ollaya ships six models:

- **Laya**: the fastest option TypeSafe trains for choice, score, and yes-no questions in 100+ languages
- **Decider**: accuracy-focused, built on Qwen3.5 architecture
- **NLI / GLiClass**: zero-shot classifiers for typed decisions without a TypeSafe-specific format requirement
- **Von**: ModernBERT-large with an 8,192-token context window, for longer inputs
- **Qwen3guard**: safety screening across 119 languages

The quality tradeoff is real and worth naming honestly. HN comments in the Ollaya thread cite Laya at 30.25 on a shared decision benchmark versus Jev (the TypeSafe hosted model) at 63.29. That is a substantial gap on harder queries — users report Laya "often makes wrong decisions with complex queries." For simple binary classifications or routing decisions on well-formed inputs, local Laya is probably fine. For more ambiguous tasks where calibration matters, the gap shows.

This mirrors the pattern we see repeatedly in the open vs. API divide: open weights trade some quality for control, latency, and cost. For the workloads where decision models actually make sense — high-throughput, low-latency, structured inputs — the quality floor may be sufficient.

## The broader point

What Ollaya represents is the infrastructure layer catching up to a conceptual layer that moved fast. TypeSafe's Jev established a paradigm: an API contract for structured decisions that is distinct from generative LLMs. Ollama established a precedent: a local-first model runner with a clean CLI and a swap-in API. Ollaya is the intersection.

If the ecosystem develops the way NLP tooling tends to — improvements in open model quality over time, infrastructure tooling growing around a common API shape — the gap between local Laya and hosted Jev will narrow. For now the tradeoff is explicit: sub-10ms local inference at 30 points of benchmark quality versus 63 points of hosted inference with API latency and per-call pricing. Both sides of that equation are usable, depending on what you're building.

The [repository](https://github.com/ollaya-dev/ollaya) is Apache-2.0 and the project is in public beta.
