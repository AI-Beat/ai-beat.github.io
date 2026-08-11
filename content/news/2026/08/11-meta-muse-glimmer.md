---
title: "Meta's Muse Glimmer: A 30B Local Agent Model"
date: 2026-08-11T06:19:38+00:00
draft: false
slug: meta-muse-glimmer-local-agent
categories: [models]
tags: [models, agents, inference, on-device, meta, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Meta's Muse Glimmer is a 30B open-weight model distilled from Muse Spark 1.2,
    targeting local agent workflows on a single consumer GPU. It slots in above
    LFM2.5-2.6B and Needle2 in the increasingly crowded on-device agent tier,
    with Apache 2.0 licensing, hybrid attention, and speculative decoding via a
    dedicated drafter model.
---

Released August 10, [Muse Glimmer](https://research.meta.ai/blog/introducing-muse-glimmer-open-agentic-model) is Meta's entry into local agent inference: a 30-billion-parameter model distilled from Muse Spark 1.2, published on Hugging Face under Apache 2.0. [LFM2.5-2.6B and Needle2](/news/2026/08/11-lfm25-needle2-edge-agents/) established the low end of this tier earlier this week — 2.5 GB and 14 MB respectively. Muse Glimmer occupies the high end of consumer-deployable: under 20 GB at 4-bit quantization, running on a single RTX or high-end Mac GPU.

## Architecture

The model has 29.6B parameters split across a 28B text decoder and a 1.8B ViT-G/14 perception encoder for images and video. The attention design in the text decoder is the notable part: it alternates between three sliding-window attention layers with a 2,048-token window (using rotary position embeddings) and a fourth full-attention layer, with that four-layer pattern repeated 13 times across 52 total layers. Grouped-query attention runs at a 16:1 query-to-key-value ratio. The result is a model that can handle a 131K context window efficiently by paying full attention only once every four layers, keeping memory costs tractable on consumer hardware.

Speculative decoding uses a companion model called DFlash — a smaller drafter that generates multiple candidate token sequences in parallel, with Glimmer verifying and accepting them. Meta reports 3.1× faster generation on an RTX 5090. This is now a standard approach for serving large models at conversational speed, but shipping a matched drafter alongside the main model from day one is practical for local deployment.

## Benchmarks

Meta reports MCP Atlas at 75.5 (compared to 54.2–62.5 for similarly-sized competitors), SWE-Bench Pro at 51.2, and Charxiv Reasoning at 78.9. The MCP Atlas number is the one to be skeptical of: it's an agentic benchmark tied to the Model Context Protocol ecosystem, and it's hard to avoid suspecting a model called Muse was optimized on workloads that overlap heavily with that evaluation suite. The SWE-Bench and multimodal reasoning numbers are more independently calibrated, and 51.2 on SWE-Bench Pro is competitive for a model this size.

The usual caveat applies: these numbers measure Glimmer against similarly-sized models, not against Muse Spark 1.2, which Glimmer was distilled from. The Hugging Face model card notes Glimmer is "generally less capable than Muse Spark overall." Quantization adds 0.2–1.0% degradation depending on benchmark.

## What it offers, and what it doesn't

Glimmer handles multimodal tool calling — using images and text together to drive structured interactions with external systems — and supports more than 100 languages. The Apache 2.0 license is a real commitment: no use-case restrictions, no commercial license fee, free to fine-tune and redistribute. llama.cpp and MLX support land on day one, so local inference on Mac hardware doesn't require additional work.

The model is not optimized for video understanding (only images and text despite the video-capable encoder inherited from Muse Spark), and the card is explicit that performance degrades for languages outside the heavily represented training set. The 4-bit quantized weights have two variants — targeting 24 GB and 32 GB VRAM systems — which means you need at least an RTX 3090 or a Mac with 32 GB unified memory to run the larger configuration comfortably.

## Where this fits

The on-device agent tier has shaped up quickly. Needle2 gets agent function-calling onto microcontrollers at 14 MB. LFM2.5 brings a full-capability agent to smartphones at 2.5 GB. Muse Glimmer brings a multimodal, multilingual agent to workstations at roughly 20 GB — closer in scale to what Ollama users run on high-end consumer hardware. The three models together cover almost the entire range from ESP32-S3 to gaming rig, which two weeks ago wasn't the case for agent workloads specifically.

The interesting design question Glimmer raises is whether multimodal tool calling at local inference speeds opens up workflows that were impractical through cloud APIs — not because the cloud API is slow, but because latency and privacy constraints at the application layer make an always-on local model more useful than a remote one you have to decide to invoke.
