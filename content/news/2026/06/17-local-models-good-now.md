---
title: "The Laptop Won"
date: 2026-06-17T06:10:00+00:00
draft: false
slug: local-models-good-now
categories: [inference]
tags: [local-inference, open-source, tooling, llm]
params:
  author: AI Beat Desk
  summary: >-
    Vicki Boykis published a careful practitioner's report on her local-inference
    stack this week, and the conclusion that stuck — ~75% of frontier model
    capability for agentic coding on a 64 GB M2 Mac — is more significant than
    the raw number suggests. The tooling layer finally grew up, and that changes
    what "running locally" means.
---

Every few months someone writes that local models are finally usable, and every few months they aren't, quite. Vicki Boykis' [post from June 15](https://vickiboykis.com/2026/06/15/running-local-models-is-good-now/) reads differently. She's not describing a demo or a hello-world; she's describing a production-adjacent workflow she uses for real tasks and has been refining for months.

Her setup: a 2022 M2 Mac with 64 GB of unified memory. Her inference server: [LM Studio](https://lmstudio.ai/). Her agent harness: Pi. Her primary models: Gemma 4 variants (`gemma-4-26b-a4b` and `gemma-4-12b-qat`), Qwen 3 MoE, and OpenAI's OSS-20B for heavier reasoning. Docker handles sandboxing. The conclusion she puts a number on — "~75% the accuracy/speed of frontier models" for agentic coding tasks like Python refactoring, type-hint linting, unit test generation, and repo bootstrapping — is her own measurement from actual use, not a vendor benchmark.

The inflection point she names is [GPT-OSS](https://openai.com/research/gpt-oss), the open-weight release that landed earlier this year. Before it, she found herself constantly dropping back to API calls to verify results. After it, the local models' failure rate dropped to the point where she didn't need to. That's a different kind of claim than "model X scores Y on benchmark Z." It's the difference between a capable model and a reliable one.

What's changed isn't only the models themselves. The tooling layer has matured considerably. LM Studio evolved from a GUI experiment into something that manages quantization selection, inference parameters, and a local OpenAI-compatible API endpoint without requiring users to compile from source or fight with CUDA drivers. The quantization side improved too: 4-bit and 8-bit representations of 26B+ parameter models have gotten meaningfully better, both in absolute quality and in the tooling for selecting the right quant level for a given task. A 26B model in Q4_K_M runs in about 16 GB of RAM on Apple Silicon and is fast enough to feel interactive.

The transparency angle in her post is worth singling out. When you run inference locally, you can watch the token stream at whatever granularity you want, adjust context window sizes mid-session, experiment with different quantization levels for the same base model, and monitor exactly how many tokens a given task consumes. With API models you're trusting vendor representations about context utilization, prompt caching behavior, and model versions. Local inference removes that indirection entirely — which matters less when you're writing a blog post and more when you're building a system that depends on consistent behavior.

There's a real ceiling too, which she doesn't hide. The 75% figure was for agentic coding tasks on her particular hardware. Complex multi-step reasoning, long-context retrieval across large codebases, and tasks that require the model to maintain coherence across many tool calls still favor frontier models substantially. The 25% gap isn't uniform — it's concentrated in exactly the places you'd expect: tasks that require sustained reasoning chains, not the pattern-matching and transformation tasks that make up the bulk of everyday coding work.

The practical takeaway is that the threshold for "when does it make sense to run locally" has shifted, and Boykis' post is a good calibration point for where it sits right now. For a developer with appropriate hardware (the 64 GB RAM ceiling is the real constraint, not the GPU), routine coding assistance that doesn't require the model's best reasoning is now plausibly a local workload. That's not about avoiding API costs — the costs are low — it's about latency, privacy, and the ability to run these tools in air-gapped or offline environments where API calls are not an option.

