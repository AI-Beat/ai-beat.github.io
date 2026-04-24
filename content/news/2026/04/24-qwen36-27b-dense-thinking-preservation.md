---
title: "Dense Beats Sparse, and Thinking Persists"
date: 2026-04-24T06:19:24+00:00
draft: false
slug: qwen36-27b-dense-thinking-preservation
categories: [models]
tags: [open-weights, coding, agents, architecture, moe]
params:
  author: AI Beat Desk
  summary: >-
    A week after Qwen3.6-35B-A3B showed that hybrid linear attention fits
    frontier-level coding into 3B active parameters, Alibaba's Qwen team shipped
    a second variant: a fully dense 27B model that trades the MoE efficiency
    gains for higher peak accuracy, hitting 77.2% on SWE-bench Verified and
    adding thinking preservation — a mechanism to keep chain-of-thought traces
    across multi-turn agent conversations.
---

[A week ago](https://ai-beat.github.io/news/2026/04/qwen36-hybrid-attention-local-coding/) we covered Qwen3.6-35B-A3B: an MoE model with 35B total parameters but only 3B active per token, a laptop-friendly footprint, and a hybrid linear-attention architecture. The interesting story was the architecture — alternating Gated DeltaNet and standard attention blocks delivering 73.4% on SWE-bench Verified at sub-notebook compute cost.

Now [Qwen3.6-27B](https://huggingface.co/Qwen/Qwen3.6-27B) lands, released under Apache 2.0 on April 22, and it complicates the story in a productive way.

## Same architecture, different density

The 27B model uses the same hybrid structure: Gated DeltaNet for linear attention layers, standard Gated Attention for full-attention layers, 262K token context extensible to ~1M via YaRN. But it's *dense* — all 27 billion parameters are active on every forward pass. There's no expert routing, no sparsity, no 3B-active shortcut.

The tradeoff this creates is real. The MoE model is cheaper to run per inference: 3B active parameters means a fraction of the compute per token. The dense model costs more per token but yields higher accuracy. On [SWE-bench Verified](https://www.swebench.com/), the dense 27B hits 77.2% versus the MoE's 73.4%. On SWE-bench Pro, it scores 53.5% versus the MoE's lower result. Terminal-Bench 2.0 goes to 59.3%, SkillsBench to 48.2%.

More telling is the storage comparison: 55.6GB for the 27B dense model versus 807GB for the old [Qwen3.5-397B-A17B](https://huggingface.co/Qwen/Qwen3.5-397B-A17B) that it outperforms across those benchmarks. That gap — 15× smaller, higher scores — is partly the generational architecture improvement and partly the fact that the 397B model's enormous parameter count was in some sense paying for something the DeltaNet hybrid can provide more efficiently.

In practice: if you're running inference at scale, the MoE variant is usually the right choice. If you want peak local performance and have 60GB of disk and enough VRAM, the dense 27B is worth pulling down. `ollama pull qwen3.6:27b` works.

## Thinking preservation

The feature worth more attention is **thinking preservation**. This is a new flag in the chat template: `{"preserve_thinking": true}`.

Standard reasoning models — Qwen3.6 included, in its default mode — generate chain-of-thought before every response, then strip that reasoning out before the next user turn arrives. The visible history shows only the final answers. The next turn starts from scratch; the model has to re-derive any context it built up in reasoning blocks from previous turns.

For single-turn tasks this doesn't matter. For multi-turn agent workflows — where the same model is iteratively editing a repository, running tools, checking outputs, and revising — it matters a lot. A coding agent that's spent three turns reasoning through an architecture might throw away useful work on turn four if it can't see its own prior analysis.

Thinking preservation keeps those reasoning traces in the conversation history. The model can attend to what it already worked out. The tradeoff is obvious: the context grows faster, which costs tokens and eventually hits the context limit. But with a 262K native window, a typical agent session has significant headroom before that becomes a constraint.

The mechanism is simpler than it might sound. There's no new learned component here — it's a template-level decision about what goes into the context window. The reasoning tokens that would have been hidden are instead formatted and kept. This is closer to prompt engineering than architecture, but it's a useful engineering decision that's been made at the model-card level so users don't have to implement it themselves.

Whether this matters in practice depends heavily on task type. For agents that are doing mostly independent tool calls where each step is self-contained, the gain will be small. For agents doing iterative work on a shared artifact — a codebase, a document, a data pipeline — keeping the reasoning state visible is likely to reduce both token waste and mistakes from forgetting earlier analysis.

The 77.2% SWE-bench Verified score already places the 27B model in a narrow tier of models that can reliably resolve real GitHub issues autonomously. The thinking preservation addition is a push to make that capability more practical in the kinds of long, stateful sessions where agentic coding actually happens.
