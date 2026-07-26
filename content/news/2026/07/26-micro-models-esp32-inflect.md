---
title: "The Clever Angles"
date: 2026-07-26T06:11:00+00:00
draft: false
slug: micro-models-esp32-inflect
categories: [research]
tags: [embedded, edge-ai, tts, quantization, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Two independent projects landed on Hacker News today pursuing the same idea from
    different directions: fit something genuinely useful into the smallest possible package.
    One runs a 28.9M-parameter LLM at 9.5 tok/s on an $8 ESP32-S3 by keeping the
    embedding table in flash. The other fits complete voice synthesis into 9.36M parameters
    that run faster than real-time on a laptop CPU. Neither is frontier work. Both are
    instructive about what constrained-environment AI actually requires.
---

Two unrelated projects surfaced on Hacker News today that are doing the same basic thing: finding the non-obvious angle that makes AI work in a space everyone assumed was too tight.

The first is [esp32-ai](https://github.com/slvDev/esp32-ai), a 28.9M-parameter language model running on an ESP32-S3 microcontroller that costs about $8. The model generates text at 9.5 tokens per second end-to-end. That number will seem either impressive or underwhelming depending on your frame of reference; on a chip with 512KB of SRAM and an 8MB PSRAM extension, it is striking. The model was trained on [TinyStories](https://huggingface.co/datasets/roneneldan/TinyStories) — synthetic short stories simple enough for small models to learn coherent narrative structure — so it writes coherent fiction but won't follow instructions or answer factual questions. It does not matter much. The point is the architecture.

The key is per-layer embeddings, a technique from Google's Gemma work. In most LLMs, the embedding table accounts for a disproportionate share of total parameters — here, 25 million of 28.9 million. The standard assumption is that you need the whole thing in fast memory during inference. Per-layer embeddings challenge that: because each token only activates a specific slice of the embedding table, you can keep the full table in flash storage and load only the rows you actually need. Each token lookup requires about 450 bytes of data pulled from the 16MB flash chip. The transformer core — the part that actually computes attention and feed-forward transformations — stays small enough to fit in SRAM.

Flash access is slow compared to SRAM, which is why this approach produces 9.5 tok/s rather than something faster. But the tradeoff is what makes it work at all: the hardware constraint (not enough fast memory for the full model) drove a design choice (put the embedding table in slower storage) that actually fits the inference pattern (you only ever need a small fraction of the embeddings per forward pass). This is the kind of solution that looks obvious in retrospect.

The second project is [Inflect-Micro-v2](https://huggingface.co/owensong/Inflect-Micro-v2), a text-to-speech model from independent developer Owen Song. It has 9,356,513 parameters and weighs 37.53MB in FP32. It runs at 6.28× real-time speed on a four-thread CPU configuration and outputs 24kHz mono audio. In a community preference study against KittenTTS and Piper (the benchmark small TTS models), it was preferred 66.2% of the time. It is Apache-2.0.

The architecture is VITS — Variational Inference with adversarial learning for end-to-end TTS — which combines a text frontend, normalizing flow, and a neural waveform decoder into a single end-to-end model. Getting VITS down to under 10M parameters without losing quality involves pruning hidden channels aggressively (192 latent channels, 96 text hidden channels, 3 encoder layers with 2 attention heads) and accepting specific constraints: English only, a single fixed male voice, no zero-shot cloning, and reduced expressiveness on unusual phrasing. Song trades features for size, which is the right trade for an edge-deployment TTS model.

The limitations matter: if you need multiple voices, accents, or speaker cloning, you're looking at a much larger model. But for a use case like a screen reader, a device that reads out notifications, or a robotics platform that needs to communicate verbally, Inflect-Micro-v2 is a working solution that fits in a GitHub repo and runs on a Raspberry Pi without a GPU.

What connects these two projects is not the application domain but the design philosophy. Both ask: what specifically can I give up? The ESP32 project gives up fast embedding access in exchange for fitting on constrained hardware. Inflect-Micro-v2 gives up speaker variety and accent coverage in exchange for extreme compactness. Neither compromise is lazy — they're the result of identifying exactly which capability is load-bearing for the target use case and then cutting everything else.

The dominant narrative about AI deployment focuses on scale: bigger models, faster chips, more memory. These two projects are not a rebuttal to that narrative — frontier models are getting better faster than small models are closing the gap. But they illustrate that a lot of practical work does not require frontier performance. A $8 microcontroller telling a story at 9.5 tok/s is not competing with GPT-5.6 Sol. It is competing with nothing, because there was nothing running LLMs on that hardware before.
