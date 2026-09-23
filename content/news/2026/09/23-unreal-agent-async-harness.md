---
title: "The Model Shouldn't Wait"
date: 2026-09-23T06:20:52+00:00
draft: false
slug: unreal-agent-async-harness
categories: [agents]
tags: [agents, infrastructure, open-source, cost, go]
params:
  author: AI Beat Desk
  summary: >-
    Unreal Labs released an MIT-licensed Go agent harness on September 22 that
    runs tools asynchronously — the model never blocks while a slow command
    executes. On Terminal-Bench 4.0 it matches Codex's pass rate at 39% lower
    cost, and beats Codex on SWE-Atlas QnA and DeepSWE 1.1. The fix is
    architectural: decouple tool execution from model turns.
---

The default implementation of a tool-calling agent is simple: model thinks, calls a tool, waits for the result, thinks again. That's how most agent harnesses work, and it's also where a substantial fraction of per-task cost comes from. Waiting on a slow shell command or file operation is time the model's context window is sitting open, racking up tokens, doing nothing.

[Unreal Labs shipped Unreal Agent](https://unreallabs.ai/blog/unreal-agent/) on September 22 — an MIT-licensed Go harness built around breaking that wait. When the model calls a tool, the harness immediately logs a record in the "in-progress" state and continues the model's turn. The tool runs in the background. When it finishes, the result is appended and the model is called again with the update. The model never blocks.

The practical consequences compound:

- The model can issue multiple tool calls before any of them resolve, then process the results as they arrive.
- User steering messages are accepted without delay, even mid-execution of a long-running task.
- Context growth from blocked wait time is reduced, which matters for per-task cost on long agentic runs.

The benchmark numbers are concrete. On Terminal-Bench 4.0, Unreal Agent running GPT-6 Astra matches Codex's 57.9% pass rate at $1,428 per task against $2,350 for Codex — 39% cheaper. On SWE-Atlas QnA it scores 65.8% versus Codex's 63.3% at 28% lower cost. On DeepSWE 1.1 it reaches 72.4% against Codex's 69.0% at 16% lower cost. Cost advantage and a small performance edge appear together across all three benchmarks, which is the signature of a structural efficiency gain rather than a tradeoff.

This connects directly to what [HarnessTax](https://harnesstax.github.io/) — an empirical study from earlier this month — established: the same model with different harnesses can differ by 32× in per-task cost on SWE-bench Lite, with context consumption accounting for more of that variance than call count. Unreal Agent addresses both. Async execution removes blocked wait time that inflates context size. Minimal, token-optimized outputs reduce per-call overhead. The HarnessTax framing was "the harness matters more than you think." Unreal Agent is an attempt to build the harness that reflects that.

There are obvious limits. Async execution helps most when tool calls are largely independent — parallel file writes, parallel searches, parallel test runs. On tasks where each tool call must read the output of the previous one before proceeding, the sequential dependency removes most of the advantage. The claim here is about cost efficiency on common coding and agent benchmarks, not a general speedup for all task types.

The distribution is three components: a [Go library](https://github.com/unreallabsai/unreal-agent) for embedding in your own agent, a runner executable for standalone use, and a Harbor-compatible benchmark runner. Provider support covers OpenAI, Codex, OpenRouter, Fireworks, and Ollama.

The design philosophy — minimize the harness overhead, run everything you can in parallel, don't make the model wait — is the right direction. Whether that design survives contact with more heterogeneous production workloads is the open question, but the benchmarks suggest the basic premise holds.
