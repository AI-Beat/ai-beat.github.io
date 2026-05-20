---
title: "The 76-Point Serving Backend Lottery"
date: 2026-05-20T06:10:00+00:00
draft: false
slug: forge-agentic-guardrails-local-llm
categories: [agents]
tags: [agents, local-llm, reliability, tool-use, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Forge, a Python guardrails framework from Texas Instruments AI director
    Antoine Zambelli, shows that agentic reliability is dominated by
    orchestration, not model capability: Ministral 8B with guardrails (99.3%)
    outperforms Claude Sonnet without them (87.2%). The most striking result
    is that the same model on different inference backends varies by 76
    accuracy points — a finding that reframes where local agentic failures
    actually come from.
---

The common explanation for why local models struggle with multi-step agentic workflows is capability: a 7B or 8B model simply isn't smart enough to chain tool calls reliably. [Forge](https://github.com/antoinezambelli/forge), an open-source Python framework presented at [ACM CAIS '26](https://www.caisconf.org/program/2026/demos/forge-agentic-reliability/), offers evidence that this explanation is wrong — or at least mostly wrong. The bottleneck isn't the model. It's the scaffolding.

Antoine Zambelli, AI Director at Texas Instruments, built Forge as a reliability layer for self-hosted LLMs doing tool-calling work. The framework adds guardrails around model invocations: retry nudges when tool calls are malformed, step enforcement when the model tries to skip steps, error recovery when a tool returns unexpected output, and VRAM-aware context compaction when the context budget fills. None of this changes the underlying model weights. It changes what happens when things go wrong.

The headline benchmark is striking: Ministral 8B running with Forge achieves 99.3% completion accuracy on multi-step agentic workflows — within one point of Claude Sonnet at 100%, and higher than Claude Sonnet without Forge (87.2%). The evaluation covered 97 model/backend configurations, 18 scenarios, and 50 runs each, which is enough to take the results seriously.

## Why Compounding Matters

The math behind this result is straightforward and tends to get underappreciated. If a model has 95% reliability on each individual tool call, then across five sequential calls the probability that all of them succeed is \(0.95^5 \approx 77\%\). That's a 23% workflow failure rate even though each individual step was quite reliable. At 10 steps it drops to 60%. The reliability problem compounds faster than intuition suggests.

Guardrails interrupt this compounding. When step 3 produces a malformed tool call, Forge's retry logic catches it, nudges the model with a correction, and tries again — rather than letting the error propagate or failing the entire workflow. The accuracy gain from this isn't primarily about recovering edge cases; it's about keeping the compound failure rate from accumulating across a long sequence.

This also explains why frontier models without guardrails fall to 87% while an 8B model with Forge reaches 99%: the frontier model is more reliable per step, but without structured recovery it still compounds errors across long workflows. Forge's value isn't in compensating for weak models; it's in making error recovery systematic.

## The Backend Surprise

The most practically important finding in Forge's evaluation isn't the headline accuracy number — it's what happens when you hold the model constant and change the inference backend.

Testing Mistral-Nemo 12B with native function calling enabled: llama-server achieved 7% workflow completion accuracy, while Llamafile in prompt mode achieved 83%. Same model, same hardware, same tasks. A 76-point spread.

The difference is in how function calling is surfaced. llama-server's native function calling mode exposes structured JSON tool calls through a dedicated interface; Llamafile in prompt mode formats tool calls as text and relies on the model's instruction-following. Forge's rescue parser handles the prompt-mode output better than the structured interface, which apparently introduces its own failure modes at the serialization boundary.

This has an immediate practical implication: if you're running local models for agentic work and your completion rates are poor, the first variable to examine isn't your model — it's your serving stack. The variation attributable to backend choice is larger in these results than the variation attributable to moving between different 8B models.

## What Forge Actually Ships

Forge exposes three ways to use the framework. The `WorkflowRunner` provides a full lifecycle manager for structured agent loops. The guardrails middleware integrates into existing orchestration systems. And there's an OpenAI-compatible proxy server that applies the reliability layer transparently — you point your existing code at the Forge endpoint instead of the model endpoint and get retry logic and context management without restructuring anything.

The VRAM-aware context budgeting is worth noting separately: it implements tiered compaction that trims context before it exceeds GPU memory limits, which matters for long-running workflows on consumer hardware where you can't simply let context grow indefinitely.

None of this is exotic engineering. Retry logic and context management aren't new ideas. What Forge did is systematically evaluate exactly how much they matter, across enough configurations to establish real numbers rather than impressions. The result — that orchestration quality accounts for most of the reliability gap between local and frontier models — is a useful correction to a common assumption.

If you're building agentic workflows on local models and treating reliability as a model selection problem, the evidence here suggests you should try treating it as an orchestration problem first.
