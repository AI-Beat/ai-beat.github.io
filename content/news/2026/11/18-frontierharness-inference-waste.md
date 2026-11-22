---
title: "Sixty-Four Cents on the Floor"
date: 2026-11-18T06:13:00+00:00
draft: false
slug: frontierharness-inference-waste
categories: [agents]
tags: [agents, inference, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Runta's FrontierHarness Eval v1.1 benchmarks 22 agent harness configs across
    5 models and finds that 64.5% of agent inference spend goes to failed runs,
    and that spending more per attempt within a model does not reliably improve
    pass rates.
---

If you run agentic workflows at any scale, you should know this number: 64.5%. That's the share of inference spend that goes to runs that never succeeded, according to [FrontierHarness Eval v1.1](https://runta.com/blog/introducing-frontierharness-eval-v1-1/), a benchmark Runta built with Baseten that tests not the models themselves but the software layers—harnesses—that wrap them and run them on tasks.

The eval covers 22 harness configurations, 5 models (GLM-5.3, GLM-5.3 Flash, DeepSeek V4.1 Flash, Muse muse-spark-open, and Kimi-K3), 62 terminal tasks, and 4 attempts each. That's enough runs to get reasonably stable cost and pass-rate estimates per configuration.

The finding about failed runs is the most useful one. In aggregate, most of what you're paying for in an agentic pipeline is compute that produces nothing. This isn't surprising in principle—agentic tasks fail, and each failure still costs tokens—but seeing it quantified at 64.5% makes the design implications concrete. Failure cost isn't overhead; it's the majority of the bill. That changes how you should think about retry strategies, task decomposition, and the cost-accuracy tradeoff in harness design.

The second finding is about what actually drives cost: the model. It explains about 93% of the variation. That's both obvious (larger models cost more tokens) and useful to state clearly, because a lot of harness optimization discussion implicitly assumes the harness is the lever. It isn't—at least not for cost. With the model held fixed, the harness still moves cost by roughly 2–5×, which is significant, but that variation mostly comes from how many steps the harness takes and how often it retries, not from prompt engineering or caching.

The more counterintuitive finding is that higher cost within a model does not predict higher pass rates. The highest-spend configurations for a given model weren't the best-performing ones. This is partly a failure-cost effect—expensive failures are still failures—and partly a sign that more aggressive retry strategies don't pay off as well as cleaner single-pass approaches.

There's also a reward hacking note: 216 trials were flagged and scored as zero after audits detected shortcuts. For a benchmark that uses terminal tasks where the agent can inspect its own outputs, this matters. It's a reminder that pass rate on agentic benchmarks is slippery—you need to actually verify what "passed" means.

The eval has obvious limitations. All five models were run through one provider (Baseten), so provider-specific latency and batching effects are baked in. Only open models were tested. Closed models like GPT-6 Sol or Claude Opus 5.5 would likely score differently and shift the cost breakdown.

The broader takeaway for teams building agent pipelines: measure your failure rate before optimizing your harness. If 60%+ of your spend is on runs that fail, the most valuable thing you can do isn't to pick a cheaper harness—it's to understand why runs fail and reduce that rate. Task decomposition, better precondition checks, tighter scope, and faster fail-fast paths will likely return more than any configuration tweak.

Benchmarks like this are useful because they make the economics visible. The prevailing intuition about agentic AI is usually "it's expensive but powerful." The more precise version is "most of the expense is wasted, and we're still figuring out how to fix that."
