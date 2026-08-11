---
title: "Two Philosophies of the Tiny Agent"
date: 2026-08-11T06:19:38+00:00
draft: false
slug: lfm25-needle2-edge-agents
categories: [inference]
tags: [inference, edge, agents, on-device]
params:
  author: AI Beat Desk
  summary: >-
    LFM2.5-2.6B and Needle2 arrived this week at opposite ends of the weight-class
    spectrum — one trimmed but architecturally orthodox, the other stripped of its
    feed-forward layers entirely — and together they define the two credible paths to
    running a real tool-calling agent on constrained hardware.
---

Getting a language model to run on a phone is not news anymore. Getting a real *agent* — something that can call tools, reason over multiple steps, and recover from failures — to run in under 3 GB on a Raspberry Pi, or in 28 MB on an ESP32-S3 microcontroller, is something else. Two projects released this week trace different paths toward the same destination.

## Liquid AI: keep the architecture, train harder

[LFM2.5-2.6B](https://huggingface.co/blog/LiquidAI/lfm2-5-2-6b) is a 2.69-billion-parameter model released August 4, pretrained on roughly 34 trillion tokens and built specifically for agent workloads. The architecture — LFM2, Liquid AI's own variant — looks broadly like a standard dense transformer, but the training pipeline is where most of the work happened. Post-training runs through four stages: supervised fine-tuning, teacher specialization, multi-domain on-policy distillation (MOPD), and finally agentic reinforcement learning run inside real agent frameworks, not synthetic traces.

The benchmark numbers are convincing for the size class. On IFStruct, a structured instruction-following test, LFM2.5-2.6B scores 85.49 against a 36–79 range for comparable smaller models. ToolSandbox puts it at 77.83 versus 52–76 for models up to 4× its parameter count. Hardware performance is equally practical: 220 tokens/second on an Apple M5 Max CPU, 113 tokens/second on an AMD Ryzen desktop, and roughly 30 tokens/second on a smartphone, all while staying under 2.5 GB of memory. At H100 serving scale it reaches nearly 15,000 tokens/second, so this is a model that works the full stack from a wristwatch to a data center.

What Liquid AI didn't sacrifice is breadth. The model handles more than 100 languages and supports 128K context, which matters for agentic tasks that involve long tool-call histories. The framing here is "agent as a first-class use case" rather than an afterthought added via prompting — the reinforcement learning stage optimized directly for multi-step completion, not for being a chatbot that can tolerate function-call syntax.

## Cactus Compute: rethink the architecture itself

[Needle2](https://cactuscompute.com/needle) started from a different premise. If the binary has to fit in 14 MB total, the standard transformer is not the right starting point. The model has 45M parameters and uses a Simple Attention Network (SAN): standard transformer feed-forward blocks are replaced with a lighter Hadamard MLP. The attention mechanism uses grouped-query attention and what the authors call "engram" key-value memory; the residual streams use sandwich normalization and gated connections. Quantization is CQ2-bit trained from the beginning — not post-training compression applied to a float32 checkpoint — which is part of why the compression is this aggressive without the model falling apart.

The result: a 14MB binary that runs a full inference session in 28 MB of peak RAM. Decode speed is 500+ tokens/second on a Raspberry Pi 5 and 300–700 tokens/second on budget phones. Cactus reports it running on ESP32-S3 microcontrollers — devices with a few hundred kilobytes of available memory and no GPU or NPU.

Needle2 doesn't do general conversation. It maps natural language to declared device functions with typed parameters, extracts structured data from documents against provided schemas, and refuses off-topic requests using a learned confidence score. Grammar-constrained decoding ensures output stays within the declared schema, which matters for reliability on device control tasks where a hallucinated parameter value can do real damage.

## What the gap reveals

The 60× parameter difference between LFM2.5 and Needle2 isn't just a size tradeoff — it reflects different theories about what "agent" actually requires. LFM2.5 bets that you need broad capability: planning over long horizons, multilingual coverage, contextual recovery across hundreds of tool calls. Needle2 bets that most devices need only one thing done reliably: a specific verb, in a specific format, with the right parameters.

What's striking about both releases is the deliberate de-emphasis of general capability. Neither optimizes for MMLU score or prose quality. Both treat the tool-calling loop — observe, plan, act, recover — as the primary task, and accept whatever other capability losses are necessary to serve that loop efficiently on constrained hardware. That's a cleaner engineering bet than trying to fit a general-purpose assistant into a small model, and the benchmarks suggest it works.

The question the two projects collectively raise is where the line is between these two modes. LFM2.5's 2.5 GB floor is reachable on most phones released in the last two years. Needle2's 28 MB ceiling means even smart home sensors are in scope. Everything in between is an open design space.
