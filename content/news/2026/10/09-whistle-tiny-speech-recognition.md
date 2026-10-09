---
title: "Seventeen Megabytes, Seven Languages"
date: 2026-10-09T06:13:00+00:00
draft: false
slug: whistle-tiny-speech-recognition
categories: [models]
tags: [speech-recognition, on-device, open-source, inference, edge]
params:
  author: AI Beat Desk
  summary: >-
    Cactus Compute released Whistle, a 16.9MB speech-to-text model that beats
    Whisper base on most benchmarks at less than a ninth of the size. It runs on
    CPU with no dependencies, targets 17 platforms including microcontrollers and
    WASI, and shares its runtime engine with the company's Needle LLM.
---

Whisper base is 145.3 MB. Moonshine tiny v2 — one of the better compact STT models around — is 41.9 MB. Cactus Compute's [Whistle](https://cactuscompute.com/blog/whistle) is 16.9 MB and beats both on most standard benchmarks.

Released on October 2, Whistle runs entirely on CPU with no dependencies, covers seven languages (English, German, French, Spanish, Italian, Dutch, and Polish), and returns word-level timestamps alongside transcripts. On an Apple M4 Pro CPU with 10 seconds of audio it hits first token in 11.1 ms at 1,319 tokens per second.

The benchmark comparison is worth sitting with: Cactus reports Whistle outperforms Whisper base on LibriSpeech test-clean despite being 8.6× smaller. It loses on TED-LIUM and AMI — likely because Whisper's training mix covers more conversational and meeting audio — but wins on LibriSpeech, SPGISpeech, Earnings-22, and the FLEURS average. The exact WER figures are in charts on the model page rather than in the text.

## Architecture

The encoder is eight Simple Attention blocks — the same module used in Cactus's [Needle](https://cactuscompute.com/blog/needle) text model — running non-causal attention over the 375 frames produced by a convolutional front-end on 30 seconds of 16 kHz mono audio (80 log-mel bins). The decoder is eight Laddered Simple Attention blocks with gated cross-attention to the encoder in every layer. Beam search with keyword biasing runs over a vocabulary of 8,192 pieces plus seven language tokens.

One deployment-oriented detail: each decoder depth was trained as its own model. This lets you select the number of active decoder layers at load time via `--audio-depth` and get a properly trained model at that depth, not a truncated one. On constrained hardware you pick the depth that fits your latency budget without retraining anything.

## Where it runs

The 17 supported targets include mobile (iOS/Android), WASI, browser via WebAssembly, and microcontroller-class platforms. WASI in particular is useful: it means Whistle can run inside any WebAssembly runtime — edge functions, sandboxed plugins, server-side WASM — without a native binary or a network call.

The bigger picture is that Cactus is building a shared runtime stack. Whistle and Needle run on the same CPU engine, so in principle the same binary handles speech-to-text and tool-calling text generation. For an embedded device that needs both speech input and local reasoning without network access, a unified inference runtime is more useful than two separate ones. It also simplifies deployment to platforms where managing multiple native libraries is painful.

Weights are on [Hugging Face](https://huggingface.co/Cactus-Compute/whistle) and source on GitHub. The page calls it "open" without naming a specific license — worth checking the model card before production use.
