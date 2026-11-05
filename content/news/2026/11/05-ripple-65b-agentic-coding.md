---
title: "The 65B That Beat the Billion-Parameter Models"
date: 2026-11-05T06:13:25+00:00
draft: false
slug: ripple-65b-agentic-coding
categories: [models]
tags: [open-source, agentic-coding, efficiency, rl, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Ricursive Intelligence released Ripple-65B on November 4, a dense 65B open-weight
    model that scores 80.52 on TerminalBench 2.1 — beating models more than ten times
    its size. The story isn't just the benchmark: it's how they got there with $32M,
    1.4M RL rollouts, and no purchased training data.
---

Ricursive Intelligence is a small company most people haven't heard of. Its main product is an AI chip design platform. On November 4 they released [Ripple-65B Preview](https://www.ricursive.com/ripple), a general-purpose open-weight model that scores 80.52 on TerminalBench 2.1 for agentic coding — placing it above Reflection Beam (501B, 80.1), Meta Muse Spark Open (816B, 76.4), and NVIDIA Nemotron 3 Ultra (550B, 56.4).

That is a dense 65B model topping models with ten to twelve times the parameter count on an agentic coding benchmark. The claim is worth scrutinizing, but the numbers are at least internally consistent and interesting enough to unpack.

## The efficiency story

The most specific technical claims aren't about the benchmark score — they're about how the model was trained.

Ripple-65B used roughly **1.4 million RL rollouts**, about 70 times fewer than Reflection Beam, the previous open-weight SOTA on the same benchmark. Total R&D cost was approximately **$32M**, which Ricursive puts at one-tenth the estimated cost of Reflection Beam's development.

The pretraining setup involved 12 trillion tokens with 1M-token context, and posttraining used no purchased data — only synthetic pipelines. For the architecture, they went with a **dense Transformer** rather than MoE: 80 layers, d_model 5120, with gated attention, global NoPE (no positional encoding), and value residuals. Reaching about 40% MFU without custom kernels or quantization is a reasonable signal of a clean implementation.

The efficiency gains the team reports come partly from systematic model laddering experiments — running smaller experiments to predict compute-optimal configurations before committing to full-scale runs. They report up to 16x better compute efficiency on code at 4B scale compared to the Olmo 3 ladder.

## Why dense, not MoE?

The comparison with MoE models is interesting. MoE architectures have dominated the last two years of large model releases precisely because they let you scale total parameters without proportionally scaling compute. A 2.8T-parameter MoE with 104B active parameters (like Kimi K3.1 Pro, also released this week) is very different from a 2.8T dense model.

Ricursive's bet seems to be that for **agentic coding tasks specifically**, a smaller but high-quality dense model with an RL posttraining pipeline optimized for that domain outperforms much larger MoE models that weren't trained as specifically. TerminalBench 2.1 tests models on agentic command-line tasks — write code, run it, fix errors, iterate — which rewards coherent long-horizon reasoning over raw parameter scale.

The fact that Ripple-65B runs locally on hardware like an NVIDIA DGX Spark matters too: coding agents that can operate on local hardware have different latency and cost profiles than API-only models.

## Caveats

All benchmark numbers are self-reported by Ricursive. TerminalBench 2.1 is a relatively narrow benchmark for agentic command-line coding tasks. The Reflection Beam cost estimate (from which the "one-tenth" figure derives) is external and unverified. The model is a preview, not a final release.

That said, the specifics — 1.4M rollouts vs. ~100M, $32M total, no purchased data, 40% MFU, 16x efficiency on laddering — are the kind of details that point to a team that actually ran these experiments rather than assembled marketing copy. And the architecture choices (dense over MoE, global NoPE, value residuals) are real design decisions with real tradeoffs.

The model is available on Hugging Face, Together AI, GMI Cloud, and OpenRouter. If independent groups reproduce the TerminalBench numbers, it becomes a genuinely interesting data point about the ceiling of dense models with focused RL training for agentic tasks.
