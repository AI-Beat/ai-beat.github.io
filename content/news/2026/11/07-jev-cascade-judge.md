---
title: "Calibration Is the Routing Key"
date: 2026-11-07T08:00:00+01:00
draft: false
slug: jev-cascade-judge
categories: [tooling]
tags: [llm-judges, evaluation, calibration, cost-optimization, inference]
params:
  author: AI Beat Desk
  summary: >-
    Hightouch's writeup on building a cascade LLM judge for identity resolution
    contains a useful engineering insight: calibration, not raw accuracy, determines
    whether a cheap model can route effectively to a more expensive one.
---

There's a pattern becoming common in production AI systems: use a cheap model for the easy cases, escalate uncertain ones to something more expensive. The idea is obvious, but making it work well is not. [Hightouch's write-up](https://hightouch.com/blog/llm-judges-identity-resolution) on building a large-scale identity resolution evaluator is a good case study in what actually matters.

The problem they were solving: checking whether merged customer identity records are over-merged — that is, whether one "identity" in their system incorrectly covers two or more real people. Human review runs at about 50 identities per hour and needs 1,000+ samples for reliable estimates. They wanted to do this at scale with LLMs.

Their approach was a cascade. A cheap, fast "decision model" called Jev runs on every identity first and returns a 0–1 confidence score for whether a condition is true. Identities whose confidence falls in the ambiguous middle band get sent to Claude Opus 5.5 for a second opinion. Everything else keeps Jev's answer.

The setup sounds reasonable. What's interesting is how they found out it doesn't work with just any cheap model.

## Calibration is the prerequisite

Hightouch tested two cheap decision models: Jev and Luna. Luna had better raw accuracy than Jev — 95.6% versus 95.7% on the easier test (nearly the same), but meaningfully better on other metrics. So you'd expect Luna to perform better in a cascade. It didn't.

The problem is that Luna's confidence was poorly calibrated. When it was wrong, it tended to still return high confidence. That means routing only the uncertain cases to Opus caught fewer of Luna's mistakes — and Luna needed a very narrow escalation band (10–90%) to even approximate Jev's cascade results, escalating only 6% of cases. Jev needed a wider band (30–90%) and escalated 22% of cases, but its errors clustered near 50% confidence, which is exactly the range that gets routed up.

The result: at similar accuracy, hybrid Jev cost about $0.88 per 1,000 identities on the easier test versus $3.86 for Opus alone — roughly 4.4× cheaper. On harder cases the gap narrowed to about 1.8×, but hybrid Jev was actually marginally more accurate than Opus alone (95.3% vs. 95.1%), which is within noise but at least doesn't regress.

## What generalizes

The key lesson is that for a cascade to work, you need a cheap model whose *uncertainty is honest*, not just whose answers are usually right. A model that says "I'm 85% sure" and is wrong 40% of the time in that bucket is useless for routing. A model that says "I'm 85% sure" and is wrong 15% of the time is exactly what you want.

This has a few implications for anyone building similar systems. First, it means you can't just grab whatever cheap model scores highest on your task and expect the cascade to work. You need to audit the calibration of that model's confidence scores on your specific distribution, which requires labeled data. Second, it means that adding more context or instructions to the cheap model can backfire: Hightouch found that adding domain context to the prompt didn't help decision models' calibration, possibly because they're tuned to give confident answers and additional framing just shifts which confident answer they give.

Third, and most useful: the routing band is a free tuning knob. If you have labeled data, you can sweep over band thresholds and find the cost/accuracy tradeoff that fits your budget. The escalation rate and the accuracy improvement are both controllable, as long as the underlying cheap model is calibrated.

The pattern isn't new — probabilistic classifiers escalating to expensive specialists have been a standard operations research idea for decades. But LLMs blur the layer between the classifier and the specialist. This writeup is a clean example of how to bring the classic insight back when both models speak natural language.
