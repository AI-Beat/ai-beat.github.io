---
title: "Nine Megabytes That Beat DeepSeek"
date: 2026-09-19T05:55:00+00:00
draft: false
slug: cactus-needle-3-edge-inference
categories: [inference]
tags: [inference, open-source, edge, mobile, on-device]
params:
  author: AI Beat Desk
  summary: >-
    Cactus Compute's Needle 3 ships as an 8–29 MB on-device model with a novel
    Laddered Simple Attention Network architecture, beating models 10× its size on
    mobile tool-calling benchmarks. Released today under Apache 2.0, it's a clear
    marker of how far specialized small models have come for automation tasks.
---

There is a quiet counter-trend to the race for ever-larger models: tiny, specialized models that run entirely on-device, return no network traffic, and do their narrow job better than frontier systems orders of magnitude larger. [Cactus Compute's Needle 3](https://cactuscompute.com/needle), released today by the YC S25 startup, is a useful marker of how far that approach has come.

The model ships as a single file between 8 and 29 MB, available in configurations from 2 to 20 layers, all deployable on hardware ranging from mobile phones to microcontrollers. It covers three tasks: tool calling (identifying the right function and filling its arguments from natural-language input), structured extraction (converting messy text into typed fields for invoices, bookings, or forms), and text embedding for local semantic search. It is not a chatbot and does not try to be.

The architecture isn't a shrunken standard transformer. Cactus describes it as a Laddered Simple Attention Network using Monarch Hadamard MLPs, GQA attention with causal convolutions, and what they call engram n-gram memory. The 121M parameter version operates with the computational efficiency of a 50M model. Weights are compressed via the company's own CQ2-bit format. A technical writeup hasn't appeared yet, so the design rationale for those choices is still opaque—but the combination doesn't look like anything already in common use.

The benchmark that will get the most attention: a 4-layer, 29M parameter variant, fine-tuned on Android automation commands, scores 62.5 on an internal tool-calling benchmark against [DeepSeek V4 Flash](https://www.deepseek.com/)'s 60.5. A 9 MB model matching a frontier cloud model on a specific task is a real result, with the caveat that the evaluation is self-reported rather than third-party, and the benchmark is narrow. Across Cactus's broader mobile tool-calling benchmark, Needle 3 claims to beat models 10× its size, and match models 2–3× its size on extraction. Those are also internal numbers, but the direction is consistent with what you'd expect from a heavily task-specialized model at this scale.

Two design choices stand out. First, the model returns an empty function list when it can't confidently match a user request to a known tool, rather than fabricating a plausible-sounding function call. For automation tasks, an empty result is recoverable: the system can ask for clarification or fall back to a different code path. An incorrect function invocation—especially one with plausible but wrong arguments—often isn't. Prioritizing precision over recall here is the right engineering call for production use.

Second, the scale compression is genuine. Sub-30 MB for a model that reliably handles tool calling is small enough to bundle inside a mobile app without rethinking the distribution model. It's small enough to run on a microcontroller. It's small enough to put inside a web browser extension without asking for elevated permissions. Those constraints open deployment scenarios that cloud-routed inference doesn't reach.

The license is Apache 2.0 and [weights are on Hugging Face](https://huggingface.co/Cactus-Compute/needle3). The full 20-layer model is around 35 MB; the smallest 2-layer variant is closer to 8 MB.

The broader point: the model field keeps producing systems designed to be general-purpose assistants, and those systems keep getting better at it. But they don't fill the niche for models that run offline, operate within a few megabytes, return results in a few hundred milliseconds, and refuse to hallucinate rather than risk a wrong action. Needle 3 is a serious attempt at that niche with a novel architecture, and the benchmarks suggest it's a competitive one.
