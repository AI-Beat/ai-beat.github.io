---
title: "Five Knobs, Sub-50ms"
date: 2026-08-22T06:45:00+00:00
draft: false
slug: nari-tts-sub50ms
categories: [inference]
tags: [inference, tts, speech, systems, latency, qwen]
params:
  author: AI Beat Desk
  summary: >-
    Nari Labs walked through five coordinated optimizations that bring
    Qwen3-TTS 1.7B to sub-50 ms p95 time-to-first-audio on a single H100,
    at $2 per million characters — against ElevenLabs at $100/M. None of
    the five changes require a new model architecture. Each targets a
    specific latency source, and the gains compound.
---

A text-to-speech system that takes 400ms to produce first audio is not streaming audio — it's a loading bar. Getting to a latency where the response feels immediate requires sub-100ms time-to-first-audio, and sub-50ms if you want headroom for network round-trips in a deployed application. Nari Labs' [engineering writeup](https://nari-labs.com/blog/qwen3-tts-speed-cost-frontier/) on pushing Qwen3-TTS 1.7B to that threshold is one of the better production ML engineering posts I've seen this year. Not because they invented new techniques, but because the writeup is honest about what actually moved the number.

The model itself is Qwen3-TTS 1.7B — Alibaba's open-weight speech model, which uses a Talker component to generate speech tokens, a Code Predictor to complete those tokens, and a Codec to convert them to audio. Three separate components with three separate compute profiles, and three separate places to lose latency.

The baseline problem is that leading silence — the gap between text arriving and audio starting — clocked in around 80ms before any optimization, just from the model generating tokens that correspond to silence before the first phoneme. The fix is mechanical: detect the onset of sustained speech in the generated token stream and begin decoding from there, rather than from the first token. This alone recovered roughly 80ms of TTFA. It's almost embarrassingly simple, which is part of why it's worth highlighting — the biggest single gain came from removing something the model was doing that wasn't serving the user.

The second and third optimizations address scheduling. The default pipeline treats the three components as sequential stages: Talker finishes before Code Predictor starts, Code Predictor finishes before Codec runs. Nari Labs exposes all three as independently schedulable tasks, then adds urgency-aware batching: requests waiting for first audio get higher scheduler priority than requests that have already started streaming. The result is that a new request doesn't get stuck behind an ongoing generation; it gets a fast path through the system for its first chunk.

The fourth is CUDA graph capture for the Code Predictor's 15-step generation loop. CUDA graphs work by recording a fixed sequence of GPU operations and replaying it as a single kernel launch, which eliminates the CPU overhead of dispatching each operation individually. Nari Labs implemented this using custom Triton kernels to handle the fixed loop structure. The gain here is harder to quantify from the writeup but is described as significant for tail latency — CUDA graph capture tends to reduce variance in addition to mean, which matters for p95 targets.

The fifth is incremental codec decoding: rather than reprocessing the full token history for each new audio frame, the system caches the transformer context and convolutional state from prior frames and only processes the new ones. This has no effect on first-audio latency but matters for streaming quality at sustained generation rates.

The result of all five together: 10 requests per second sustained throughput on a single H100 SXM, sub-50ms p95 TTFA, at approximately $2 per million characters. ElevenLabs charges $100/M. Cartesia charges $49/M. The 50× cost gap is partly model scale (1.7B vs. larger proprietary models) and partly inference efficiency — the writeup is explicit that the optimizations drive the cost down, not just the latency.

What's instructive about this as a case study is the sequencing. None of the five changes are exotic. Dynamic silence removal is signal processing. Unified scheduling is standard systems work. CUDA graphs are a known technique with documented APIs. Incremental decoding is basic caching. The contribution is identifying which five things matter, in this specific pipeline, and making them work together. The silence removal gain of ~80ms would be meaningless without urgency-aware batching keeping the scheduler from starving new requests; CUDA graph capture helps tail latency but doesn't help if leading silence is eating the budget.

ML infrastructure posts often focus on "we trained X" or "we quantized X" — the model change. Posts about "we made the existing model much faster and much cheaper without changing the model" are rarer and usually more useful to people actually running production systems.

The Qwen3-TTS model is [publicly available](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-Base) under Apache 2.0, so these optimizations are in principle reproducible on any H100-class system.
