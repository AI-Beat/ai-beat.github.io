---
title: "The Serving Stack Writes Itself"
date: 2026-05-10T06:10:01+00:00
draft: false
slug: vibeserve-llm-serving-agents
categories: [agents]
tags: [inference, systems, agents, paper]
params:
  author: AI Beat Desk
  summary: >-
    A University of Washington paper shows a multi-agent loop that generates
    complete LLM serving systems end-to-end. On standard workloads it matches
    vLLM; on six specialized scenarios — hybrid architectures, streaming ASR,
    constrained decoding, multimodal pipelines — it beats it by 1.7× to nearly
    6×. The paper surfaces a practical claim: the general-purpose serving stack
    is a compromise, and specialization can be automated.
---

There is a quiet assumption baked into how the industry thinks about LLM inference infrastructure: you pick a battle-tested serving system (vLLM, SGLang, TensorRT-LLM), tune it for your hardware and model, and that's the stack. Specialization happens around the edges — quantization configs, batching strategies, cache settings — but the core serving logic is fixed.

[VibeServe](https://arxiv.org/abs/2605.06068), a paper out of the University of Washington, asks whether an AI agent could write a better serving stack for you from scratch. The answer, on several important workloads, is yes.

## The Architecture

The system uses two nested loops. The outer loop acts as a planner and search tracker: given a target model and benchmark workload, it maintains a design space of candidate serving approaches and decides what to try next. The inner loop does the implementation work, using three specialized sub-agents — Implementer, Accuracy Judge, and Performance Evaluator — to write code, verify it against correctness criteria, and measure throughput or latency on the target benchmark.

The agents are instantiated using Codex CLI and have access to a skills library: accumulated serving-systems knowledge encoded from existing inference engines. This library functions as a domain-specific retrieval store for the agents, providing building blocks (KV cache management strategies, speculative decoding implementations, quantization routines) they can compose rather than invent from scratch.

On a standard deployment — Llama-3.1-8B on an H100 — the generated system reaches near-parity with vLLM and beats SGLang by about 5% on throughput. That's the baseline: automated code generation that, in the common case, works about as well as hand-tuned infrastructure.

## Where It Gets Interesting

The results on non-standard workloads are what give the paper its punch. Six scenarios, six wins:

**Code editing with predicted outputs** (Qwen3-32B): When users supply predictions of what the output should look like, a speculative decoding scheme that exploits those predictions achieves a **5.95× speedup**. A general-purpose serving stack doesn't know to look for user-supplied predictions; it optimizes for the average case.

**Hybrid-architecture prompt caching** (Olmo-Hybrid-7B): Models that mix attention and state-space layers (like Mamba hybrids) have cache management requirements that differ from pure-attention architectures. The generated system handles this correctly and reaches **3.45× throughput** over the general baseline.

**Streaming speech recognition** (Moonshine): Per-stream encoder cache management — storing and reusing encoder outputs across streaming frames — delivers **1.69× lower latency**. Standard stacks aren't structured to exploit the stateful nature of streaming ASR.

**Constrained JSON decoding** (MacBook): Grammar-constrained decoding, where the model must produce valid JSON according to a schema, benefits from specialized rejection sampling. **2.6× speedup** on Apple Silicon.

**Multimodal image generation** (Show-o2 on H100): Mixed-modality generation pipelines that interleave text and image tokens need custom scheduling. **21.4% speedup.**

**Multimodal on MacBook** (MLX): The biggest win — **6.27×** — comes from code that exploits MLX's unified memory model and Apple Silicon's specific memory bandwidth characteristics. A general-purpose stack running on Metal simply doesn't know about these.

## The Practical Implication

The paper frames its contribution as a shift from "runtime generality" to "generation-time specialization." Instead of building one stack that runs reasonably well on everything, you generate a stack that runs well on your specific workload, then run that. The cost of the generation step (agent compute, benchmark evaluation) is paid once per deployment configuration.

The significant gains tend to cluster around cases where the serving stack needs architectural awareness of something the model or workload brings: hybrid architectures, user-supplied context, platform-specific memory models, structured output constraints. These are exactly the cases that are awkward to handle in a general framework, because supporting them cleanly requires choices that interact badly with the framework's other abstractions.

What's not in the paper: long-term maintenance of the generated stack as models and workloads evolve, correctness under adversarial inputs, or reliability at production scale. The experiments are synthetic benchmarks on specific hardware. Whether the agentic loop produces systems that hold up in real deployments is an open question.

But the existence proof is interesting. If the skills library encoding inference-engine knowledge is good enough that agents can compose it into competitive serving stacks, then the work of inference optimization starts to look more like curating a knowledge base than writing low-level C++. That's a different division of labor than the field has operated on.
