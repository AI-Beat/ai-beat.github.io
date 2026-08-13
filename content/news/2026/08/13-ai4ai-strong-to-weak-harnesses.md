---
title: "Scaffolding Beats Reasoning"
date: 2026-08-13T06:23:00+00:00
draft: false
slug: ai4ai-strong-to-weak-harnesses
categories: [research]
tags: [llms, reasoning, distillation, inference, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    A preprint from Salesforce and UIUC shows that a strong model can construct
    inference-time "harnesses" — structural scaffolds applied to a weaker model
    at test time — that nearly double Theory-of-Mind accuracy from 0.49 to 0.91
    without any fine-tuning. The mechanism isn't more reasoning; it's converting
    unstable reasoning into deterministic code, routing, and format enforcement.
---

A [preprint posted to arXiv yesterday](https://arxiv.org/abs/2608.12307) from researchers at Salesforce AI Research and UIUC proposes an unusual route to improving weak models: instead of retraining them or prompting them to think harder, have a stronger model construct a structural "harness" they can use at inference time.

The setup is simple. A stronger "builder" model is given 5% of a benchmark's validation set and iteratively refines a harness — a structured scaffold that gets applied whenever the weaker "target" model generates a response. The target model itself is never fine-tuned; it just runs through the harness each time. The refinement loop is repeated across multiple rounds using the builder's own assessment of harness quality.

On four Theory of Mind benchmarks, this more than doubled performance: average target-model accuracy went from 0.49 to 0.91. That's a substantial jump for a technique that requires no training and only a modest number of refinement rounds from the builder.

What's more interesting than the number is the mechanism. The authors found that most of the gains came from three things: converting the model's unstable reasoning into deterministic code that handles specific intermediate steps, routing inputs to different processing paths based on question type, and enforcing strict output formats. They specifically ruled out "more reasoning" as the driver — longer chains of thought and broader sampling didn't explain the improvement. The scaffolding did.

Theory of Mind is a natural test case for this. ToM tasks require tracking beliefs across multiple steps (who knows what, and when), where errors tend to compound as the model tries to hold more state in generated tokens. Converting parts of this to code makes sense as a strategy: code is reliable in ways that probabilistic token generation isn't. The harness takes the parts of the task where model uncertainty accumulates — multi-step state tracking, consistent output parsing — and replaces them with deterministic procedures.

The practical implication: there's a category of tasks where weaker models underperform not because they lack knowledge but because the inference structure works against them. A strong model can identify those structural failure modes and build around them without touching the target model's weights. The builder model is functioning less like a teacher (in the distillation sense) and more like an engineer who redesigns the interface to reduce operator error.

This has a mildly uncomfortable implication for evaluation. If a strong model can construct harnesses that nearly double ToM accuracy on a weaker target model without any actual capability improvement in that model, then benchmark scores measured without harnesses may systematically understate what's achievable in deployment. Scaffolding is exactly the kind of intervention that gets deployed in production (structured prompts, routing logic, output parsers) but typically isn't in benchmark evaluations. The gap between the two just got harder to ignore.

The paper is a preprint and the results are specific to Theory of Mind tasks. ToM has a relatively clean structure — multiple-choice belief tracking over short passages — that's well-suited to code offloading. Whether the approach generalizes to more open-ended tasks, where the problem structure is less amenable to deterministic substitution, is the obvious next question. The authors note that builder model quality consistently correlated with harness quality, which suggests the ceiling scales with what you can afford to run as the builder — a useful property if it holds more broadly.
