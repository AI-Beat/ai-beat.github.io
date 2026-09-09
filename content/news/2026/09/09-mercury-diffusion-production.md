---
title: "A Thousand Tokens per Second"
date: 2026-09-09T06:11:33+00:00
draft: false
slug: mercury-diffusion-production
categories: [inference]
tags: [inference, diffusion, latency, production]
params:
  author: AI Beat Desk
  summary: >-
    InceptionLabs released Mercury 2.5, a diffusion language model doing 1,107
    tokens per second on standard NVIDIA GPUs at $0.20/$0.75 per million
    tokens. For voice agents and real-time coding assistants, the latency
    difference between autoregressive and diffusion approaches has become
    concrete and measurable.
---

[Mercury 2.5](https://www.inceptionlabs.ai/blog/introducing-mercury-2-5), released yesterday by InceptionLabs, is a diffusion language model hitting 1,107 tokens per second on standard NVIDIA GPU infrastructure. At launch pricing ($0.20/$0.75 per million tokens input/output), it's positioned as production infrastructure for latency-sensitive applications — specifically voice agents and real-time coding workflows.

The diffusion LLM architecture that Inception has been developing doesn't generate tokens autoregressively, left to right. Instead it starts with a noisy sequence and denoises the entire output in parallel steps. The theoretical advantage has always been clear: if you can predict multiple tokens simultaneously rather than one at a time, throughput should scale differently than with transformer-based autoregressive decoding. The question has been whether the practical implementation — particularly the quality and controllability you need for production use — would actually get there.

The production metrics Inception cites are more persuasive than benchmark numbers. [OpenCall](https://opencall.ai), a voice agent platform, reports their P99 response time dropped from several minutes to one second after switching to Mercury 2.5. That's not a marginal improvement; it's the difference between a voice interaction that feels broken and one that feels real. [Augment Code](https://www.augmentcode.com) reports 82% latency reduction in context compaction tasks. These are the kind of numbers that move buying decisions.

Mercury 2.5 has a 260K token context window and supports parallel tool calls and schema-aligned JSON output — which matters for agentic workflows where you're orchestrating multiple tools simultaneously. The "tunable reasoning" feature is interesting: you can dial how much of the inference budget gets spent on chain-of-thought before the final answer, which lets you make explicit latency/quality tradeoffs at call time rather than choosing a model tier.

The 40% intelligence increase Inception claims over Mercury 2 puts it "comparable to cost-optimized frontier models" — which in 2026 means roughly the o4-mini / Gemini 3.8 Flash tier. That's not leading-edge reasoning, but it's plenty for the applications where the speed matters most: voice agents responding to real-time input, coding assistants doing continuous context management, classification pipelines processing high volumes.

The interesting structural question the diffusion LLM trajectory raises is whether the architecture distinction — autoregressive vs. diffusion — will matter at the frontier. Right now diffusion is competitive on cost and speed at a capability level somewhat below the top of the autoregressive stack. Mercury 2.5 gets you to the "good enough for production" zone. Whether diffusion-based architecture can close the remaining quality gap at longer-context, harder-reasoning tasks is a separate question Inception hasn't answered yet.

But for the specific workloads where Mercury 2.5 is positioned, "fast enough and good enough" is what matters. The voice agent number — P99 from minutes to one second — isn't a benchmark. It's an application that was essentially broken and is now functional.
