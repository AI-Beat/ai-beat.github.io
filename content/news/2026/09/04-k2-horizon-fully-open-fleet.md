---
title: "Open All the Way Down"
date: 2026-09-04T06:15:00Z
draft: false
slug: k2-horizon-fully-open-fleet
categories: [models]
tags: [open-source, models, agents, MoE]
params:
  author: AI Beat Desk
  summary: >-
    MBZUAI's Institute of Foundation Models releases K2 Horizon: six models from 0.9B to 375B, all Apache 2.0 with weights, code, training data, logs, and intermediate checkpoints. The gap between this and most "open" releases is wide—and IFM even self-audited a benchmark correction before publication.
---

The phrase "open source AI" has been stretched to cover a lot of territory—weights-only drops with no training code, data cards with no actual data, and licenses that quietly bar deployment at scale. IFM's [K2 Horizon](https://ifm.ai/blog/k2) does something less common: it opens the entire stack.

[MBZUAI's Institute of Foundation Models](https://ifm.ai/) released K2 Horizon on September 3rd—a fleet of six models from 0.9B to 375B parameters, all under Apache 2.0. That license piece is table stakes; the more interesting part is what else they're shipping. For each model: weights, architecture code, training code, data or reproducible data-construction recipes, training configurations, fine-grained logs, evaluation scripts, and intermediate checkpoints across the training lifecycle. When IFM says open, they mean something closer to the FSF definition than the PR department's.

The size range is deliberate. The 0.9B model targets heavily constrained hardware—watches, glasses, edge microcontrollers. The 3.7B and 7B are positioned for on-device phone deployment. At 32B you hit the everyday heavy-use tier. The 36B model (K2-Horizon-36B-A4B) is the technically interesting mid-range: it pairs Mixture-of-Value-Attention (MoVA), a sparse attention variant, with a MoE feed-forward layer, keeping roughly 4B parameters active per token while fitting in a smaller memory footprint than the weight count suggests. The 375B flagship (K2-Horizon-375B-A23B) runs about 23B active per token with a 512K-token context window—explicitly aimed at long-horizon agentic workloads, which is the harder regime for sparse models to maintain coherence across.

Benchmark numbers here come with caveats, but an interesting one. IFM initially reported 70.2% on TerminalBench for the 375B. Before publication they ran a self-audit for reward hacking and restated the figure to 66.9%. That's an unusual move: voluntarily publishing lower numbers on your own press release. It doesn't make the score independently confirmed—it isn't—but it signals something about how the team is thinking about scientific integrity in an environment where benchmark manipulation is routine. The 0.9B model reportedly breaks 48 on AIME 2026, which would be remarkable at that scale if it holds under independent evaluation; the 7B SWE-bench figure of 82 is explicitly disclaimed by IFM themselves.

Each model was pretrained on roughly 20 trillion tokens, about half synthetic, and post-trained on more than 100 million generated tasks. The data construction recipes ship alongside the models where redistribution licenses permit. That synthetic data fraction will be important for anyone trying to understand why the small models behave as claimed—the mechanism is at least reproducible in principle.

IFM isn't a major American lab. MBZUAI is based in Abu Dhabi and has been systematically building a research program oriented toward genuine openness. K2 Horizon is the most visible output of that orientation so far. What makes it interesting isn't just the scale—the 375B landing in the same week as GPT-6 Astra is a footnote in raw capability comparisons—but the bet that full openness is a meaningful differentiator when everyone else is holding data and training code back. The intermediate checkpoints alone are worth more to the interpretability and mechanistic understanding communities than most benchmark claims.

Whether the numbers hold up under independent reproduction is the real test. The material is there to try.
