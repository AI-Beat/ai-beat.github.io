---
title: "GLM-5.2: Open Weights, Confirmed Benchmarks"
date: 2026-06-18T06:04:24+00:00
draft: false
slug: glm-5-2-weights-delivered
categories: [models]
tags: [open-source, coding, long-context, zhipu, moe, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Z.ai shipped the MIT weights for GLM-5.2 on June 17 — 753B MoE, 40B active,
    1M context — and the benchmarks back up the release: 74.4% on FrontierSWE,
    81% on Terminal-Bench 2.1, and top of the Artificial Analysis open-weights
    leaderboard. The catch is token consumption nearly double its nearest
    open-weights competitors.
---

Four days ago this site [noted that GLM-5.2 launched to Z.ai's Coding Plan subscribers with zero benchmarks](/news/2026/06/glm-5-2-ships-access-before-evidence/) — distribution before evidence, capture the installed base first and publish the numbers when they're ready. The weights were "coming next week." They're here now, under MIT, with benchmarks attached.

Z.ai released the full weights and benchmark disclosure for [GLM-5.2](https://artificialanalysis.ai/articles/glm-5-2-is-the-new-leading-open-weights-model-on-the-artificial-analysis-intelligence-index) on June 17 — no commercial restrictions, no hosting carve-outs, no fine-tuning limitations. The weights are on Hugging Face and ModelScope.

The evidence holds up.

GLM-5.2 is a 753-billion-parameter mixture-of-experts model with 40 billion active parameters per forward pass. The headline feature is a 1-million-token context window, which sounds routine in 2026 except that actually doing it without degraded performance requires architectural work. The mechanism here is called **IndexShare**: rather than instantiating a separate indexer per sparse attention layer, GLM-5.2 shares a single lightweight indexer across every four layers. The result is a 2.9× reduction in per-token compute at 1M context length. The model also adds Multi-Token Prediction layers — predicting multiple tokens ahead simultaneously — which Z.ai says raises token acceptance rates by about 20%.

On benchmarks, [Artificial Analysis](https://artificialanalysis.ai/articles/glm-5-2-is-the-new-leading-open-weights-model-on-the-artificial-analysis-intelligence-index) puts GLM-5.2 at 51 on the Intelligence Index v4.1, the highest score among open-weights models, clearing MiniMax-M3 (44) and DeepSeek V4 Pro (44). On FrontierSWE — the harder, longer-horizon successor to SWE-bench that requires agent autonomy rather than scaffold-assisted patching — it posts 74.4% against GPT-5.5's 72.6%. Terminal-Bench 2.1, which tests autonomous shell agents in live environments, hits 81%, reportedly the first open-weight model to clear 80% on that benchmark. GPQA Diamond, the graduate-level scientific reasoning test, comes in at 89%.

The caveat worth flagging is token consumption. GLM-5.2 generates roughly 43,000 output tokens per Intelligence Index task, 37,000 of which are internal reasoning chain. Comparable open-weights models run at roughly half that figure. At Z.ai's first-party pricing of $4.40 per million output tokens, the extended thinking overhead adds real cost: Artificial Analysis estimates the per-task expense at $0.46, compared to $0.18–$0.31 for competitive models. For interactive work where you need the model's full capability on a hard problem, this is acceptable. For high-volume agentic pipelines, the math changes fast.

This token-heaviness is a pattern in reasoning-focused models — the chain of thought is doing genuine work — but GLM-5.2 is notably more pronounced about it than its current open-weights peers. It is the most capable text-only open-weights model available right now, and it costs correspondingly more to run.

The clean MIT license is worth emphasizing in the current context. Earlier this month, [MiniMax M3](/news/2026/06/minimax-m3-sparse-attention/) launched with "open-weight" branding that came with meaningful usage constraints. GLM-5.2's MIT release carries none of those asterisks. For teams evaluating whether to build against open weights or pay for closed API access, the licensing clarity matters as much as the capability numbers.

The open-weights coding model space in mid-2026 is genuinely competitive — [Kimi K2.7-Code](/news/2026/06/kimi-k2-7-code-reasoning-efficiency/) shipped last week with full benchmark disclosure at launch, [MiniMax M3](/news/2026/06/minimax-m3-sparse-attention/) arrived with a 1M context and sparse attention story of its own. Multiple models from multiple geographies, different cost/capability trade-offs, real benchmark disclosure becoming the norm rather than the exception. GLM-5.2 has earned the top spot on the leaderboard. The question is whether 2.9× compute efficiency at long context translates to applications that actually need a million-token window, or whether the token overhead means most users are better served by a faster, cheaper model for 90% of tasks.
