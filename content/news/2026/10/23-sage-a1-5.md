---
title: "Sage-A1.5: Open-Weight Long-Horizon Agent from Shanghai AI Lab"
date: 2026-10-23T07:30:00+01:00
draft: false
slug: sage-a1-5-open-weight-moe-agent
categories: [models]
tags: [open-source, agents, moe, qwen, shanghai-ai-lab]
params:
  author: AI Beat Desk
  summary: >-
    Shanghai AI Lab dropped Sage-A1.5 under Apache 2.0: a 35B MoE model
    (3B active parameters) built on Qwen3.6 with parallel subagent
    orchestration and mid-task verification. It's positioned for long-horizon
    agentic tasks and open scientific discovery, with 256K context and 26
    quantized variants available.
---

There's a quiet pattern in how Shanghai AI Lab ships models: they're rarely first on leaderboards, but they reliably fill gaps that matter for practitioners. [Sage-A1.5](https://arxiv.org/abs/2610.28578), published on arXiv October 20 with [weights available on Hugging Face](https://huggingface.co/InternScience/Sage-A1.5) under Apache 2.0, is another example of that approach.

The model is a 35B Mixture-of-Experts with only 3B active parameters per token, based on Qwen3.6 and the team's previous Agents-A1. The architecture tag is `qwen3_5_moe`, and the native context window is 256K, extendable to 512K with YaRN. Twenty-six quantized variants are provided, covering llama.cpp, Ollama, LM Studio, and Jan.

The main additions over A1 are architectural choices about how the agent manages itself during long tasks rather than tweaks to the underlying transformer:

**Parallel subagent orchestration.** Rather than a single agent context growing unboundedly, Sage-A1.5 delegates subtasks to subagents running in parallel with separate contexts. The stated goal is to reduce interference between concurrent threads of work. This is a sensible design choice for tasks where subtasks are largely independent—literature review, multi-file code changes, parallel data processing—though it comes with the usual distributed-agent coordination overhead.

**Mid-task verification and strategy revision.** The model can assess whether a line of investigation is producing useful evidence and revise its plan before finishing. "Verification" here doesn't mean formal proof; it means checking intermediate results and backing out of unproductive search paths. This addresses one of the failure modes that makes current agents frustrating on longer tasks: they commit to a bad strategy and follow it all the way to an unhelpful conclusion rather than cutting their losses.

**Exploratory scientific research mode.** The paper frames a third use case around open-ended research with inspectable artifacts. The weights come with an access token system at `discovery.intern-ai.org.cn` for compute, which adds some friction over fully self-hosted runs, but the weights themselves are unrestricted.

On published benchmarks, [secondary sources](https://aiweekly.co/alerts/sage-a15-moe-agent-hits-817-browsecomp-612-swe-bench-pro) put it at 81.7% BrowseComp and 61.2% SWE-Bench Pro. For reference, BrowseComp is a hard web research task where it trails the current frontier (Kimi-K3 at 91.2%), but it's a meaningful step above A1's 75.5% on the same task. SWE-Bench Pro isn't in the arXiv abstract, so treat the specific numbers as community-sourced until the full paper PDF confirms them.

What's worth paying attention to is the efficiency profile: 3B active parameters from a 35B pool, running on commodity hardware via quantization, with a context window large enough for real document-level work. The bet here is that a well-designed routing strategy can give you agent-appropriate performance without the serving cost of a dense 35B model or the infrastructure demands of something like [Reflection's Beam](https://huggingface.co/reflectionai) (501B/23B active).

Whether the architectural additions to orchestration and verification actually help on tasks that matter—versus making a paper look good on cherry-picked demos—is the thing to watch. The weights are there to reproduce.
