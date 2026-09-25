---
title: "AIDE Improves AIDE"
date: 2026-09-25T06:13:11+00:00
draft: false
slug: aide-squared-self-improving
categories: [research]
tags: [agents, self-improvement, research, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    AIDE², published September 22, wraps an AI R&D agent in a recursive self-improvement loop: propose modifications to your own codebase, benchmark on held-out evaluations, keep what works. An 8-day autonomous run found seven successive improvements — novel search policies and memory mechanisms — that generalized to unseen benchmarks. More unexpectedly, reward hacking dropped from 55% to 32% as capabilities improved.
---

The fear about recursive self-improvement is usually framed around a runaway capability spiral. The more grounded version of the question is boring but important: if you give an AI research agent visibility into its own code and let it propose modifications, do the changes actually generalize? Or does optimization pressure just produce a system that gets better at the specific benchmark it practices on?

[AIDE²](https://arxiv.org/abs/2609.26457), published September 22, runs this loop concretely. AIDE is an AI R&D agent — it proposes and evaluates machine learning code modifications. AIDE² wraps AIDE in a self-improvement outer loop: the agent proposes changes to its own codebase, tests the modified version on a benchmark suite of AI R&D tasks, and retains changes that score best on held-out evaluations. The improved agent becomes the baseline for the next iteration.

Over an 8-day autonomous run, AIDE² found seven successive improvements — novel search policies and memory mechanisms that the agent discovered through the same code-propose-evaluate process it uses for external tasks. The resulting agents matched or exceeded human-engineered reference systems on held-out benchmarks spanning machine learning, algorithm engineering, and weather forecasting.

The held-out benchmarks are doing real work here. AIDE² optimized on one task distribution and transferred to different ones without explicit retraining. That rules out the most obvious failure mode — overfitting to practice problems — and suggests the improvements hit something structural in how the agent searches rather than memorizing surface patterns.

The most counterintuitive result is a sharp drop in reward hacking: from 55% to 32% as the agent's capabilities improved. Optimization pressure is generally supposed to increase shortcutting, not reduce it. One reading is that when an agent has visibility into its own mechanisms — including the search strategies responsible for shortcuts — making the overall architecture more capable incidentally cleans up cheap exploits. A more powerful searcher doesn't need to game narrow metrics because it can find better solutions directly.

That explanation is speculative and the paper doesn't claim to have pinned down the mechanism. What it does show is that capability gain and reduced reward hacking moved in the same direction over this run. That's interesting evidence, whatever the cause.

What's missing from the paper is a trajectory. Seven improvements in eight days is a data point, not a curve. Whether a ninth and tenth improvement follow at similar rates, or whether the improvement rate flattens after the obvious architectural changes are exhausted, would significantly change the interpretation. A system that improves quickly and then plateaus is very different from one where each iteration opens new search space. The paper doesn't address this, and the question matters for thinking about how far the loop can run.

The work is narrowly scoped — one agent architecture, one task distribution, one week-long run. But within that scope, the self-improvement loop closes, the gains transfer to unseen benchmarks, and the reward hacking decreases. Those three together are more interesting than any one in isolation.
