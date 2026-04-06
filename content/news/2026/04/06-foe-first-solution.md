---
title: "The First Guess Is Usually Right"
date: 2026-04-06T06:22:00+00:00
draft: false
slug: foe-first-solution-reasoning
tags: [reasoning, inference, efficiency, llm]
params:
  author: AI Beat Desk
  summary: >-
    A new preprint identifies a consistent pattern in large reasoning models:
    the first generated solution outperforms later alternatives, and continued
    reasoning can actively degrade accuracy. The proposed fix, called RED,
    improves performance by up to 19% while cutting token usage by 37–70%
    versus competitive baselines. It's a useful challenge to the assumption
    that more inference compute is always better.
---

There's a comforting assumption embedded in the test-time scaling literature: that generating more candidate solutions and spending more tokens on reasoning is almost always better. [A preprint posted April 3](https://arxiv.org/abs/2604.02967) challenges this directly.

The paper, called FoE (Forest of Errors), studies what actually happens inside long chain-of-thought traces in reasoning models like DeepSeek-R1. The core finding is what the authors call "The First is The Best": the first generated solution in a reasoning trace is consistently the best, and subsequent alternative solutions don't just fail to improve things — they tend to make performance worse.

The mechanism they propose is error propagation. Once a reasoning path takes a wrong turn, subsequent steps compound the error. The metaphor of a "forest" refers to the branching structure of mistakes: errors aren't isolated, they cascade and branch, forming tree-like dependency structures where correcting a downstream error requires first undoing an upstream one. By the time you're generating a third or fourth solution attempt, you're often building on a corrupted context rather than starting fresh.

This matters practically. Sampling more solutions and using majority voting or best-of-N selection is a standard strategy for improving reasoning model performance, but FoE suggests the marginal solution is often pulling in the wrong direction. The first guess, which is also the most direct, captures the model's best uncontaminated reasoning.

The paper's proposed remedy is RED: Refine and Discard. Two steps. First, the "Refine" component focuses on improving the first solution rather than generating alternatives — the intuition being that the first attempt is already in the right neighborhood and can be sharpened. Second, the "Discard" component prunes subsequent solution attempts using a dual-consistency check: if a later solution is inconsistent with two consistency criteria, it gets dropped rather than aggregated.

Importantly, RED requires no external verifier, no reward model, no separate judge — it's self-guided using the model's own consistency signals. On five benchmarks and six backbone models, RED achieves gains of up to 19% over eight competitive baselines while reducing token usage by 37.7% to 70.4%. The trade-off is favorable: you get better answers and spend fewer tokens doing it.

There are caveats worth noting. "Up to 19%" is a maximum; the typical gain across benchmarks and models isn't specified in the abstract, and that number can conceal wide variance. The paper is a preprint without affiliation, and the specific benchmarks and backbone models aren't named in the abstract, making it hard to evaluate the experimental scope from the outside. The result will need replication.

But the broader pattern is worth taking seriously. The test-time scaling story has been told mostly as a monotone: more compute, more solutions, better answers. FoE is pointing at a non-monotonicity — a regime where continued generation actively hurts. This is consistent with other findings about reasoning model pathologies (e.g., models "thinking themselves into" wrong answers by overthinking), and it has direct engineering implications for anyone building inference pipelines.

The practical takeaway, if the result holds up: don't use best-of-N sampling uncritically with reasoning models. Your first sample may already be better than the average of three. Refining that first sample is likely more efficient than generating alternatives. And the right strategy for improving accuracy might be to invest compute in sharpening one answer rather than spreading it across many.

That's a meaningful reframe of how to use these systems efficiently — not just more tokens, but the right tokens, in the right order.
