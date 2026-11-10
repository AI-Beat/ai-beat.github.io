---
title: "The Agent Workhorse at $0.10 per Million"
date: 2026-11-10T06:14:00+00:00
draft: false
slug: muse-ember-meta-agent-model
categories: [models]
tags: [meta, models, efficiency, agentic, inference]
params:
  author: AI Beat Desk
  summary: >-
    Meta's Muse Ember, released November 9, is Meta's smallest model at
    $0.10/M input tokens. It beats Claude Haiku 5.5 on most agentic
    benchmarks and costs roughly 17x less per task than Muse Spark 1.3,
    which shifts the economics for teams building agent pipelines.
---

Meta released [Muse Ember](https://research.meta.ai/blog/muse-ember) (`muse-ember-1.0`) yesterday. It's their smallest model, priced at $0.10 per million input tokens and $0.50 per million output tokens — below Claude Haiku 5.5 on cost and roughly matching or beating it on the agentic benchmarks Meta publishes.

The comparison to Haiku 5.5 is deliberate. Meta positions Ember explicitly for agent loops: tool calls, routing, extraction, long document parsing, the repetitive plumbing work that runs in the inner loops of multi-step pipelines. For that workload, benchmark scores on agent-specific tests matter more than scores on general reasoning or coding evals.

Meta's internal numbers show Ember leading Haiku 5.5 on AutomationBench-AA (37.8% vs 13.9%), Terminal-Bench 4.0 (40.9% vs 39.2%), Coding Agent Index-AA (46 vs 37), and OSWorld 2.1 (69.5% vs 68.9%). Haiku 5.5 leads on FrontierCode 2.0-AA (34.9% vs 27.2%). These are vendor-reported numbers, which means benchmark selection and methodology are Meta's choices. Independent evals will tell a more complete story.

The figure that matters most for most users isn't per-token pricing but per-task cost. Meta's calculation using the Artificial Analysis task set puts Ember at about $0.14 per task versus $2.39 for Muse Spark 1.3 — a 17x difference. If you're running agentic pipelines where each step involves several tool calls and a few thousand tokens, that ratio compounds fast.

The technical note in the blog post says Ember is the first model trained using Muse Wonder Full as its teacher — Meta's application of distillation, where a larger model's outputs shape a smaller model's behavior. This is the most plausible explanation for how they get competitive agentic performance at this price point. The model supports a 1M token context window with up to 512K output tokens and takes text, image, audio, and video inputs natively.

Third-party signals are limited so far. Cloudflare reported 30% faster p95 latency at the tail versus a comparable model, and You.com measured a 2.2s p95 on coding tasks. Both are Meta launch partners, so these aren't fully independent data points, but they're at least concrete numbers from people who actually ran it.

What's notable is that this is roughly the second time in a month that a capable agentic model has landed at or below $0.20/task. The practical effect is that multi-step pipelines, which a year ago required careful justification per use case because of cost, are starting to look affordable for routine work — the kind where every document that enters a system gets routed, parsed, and tagged by an agent rather than a hand-crafted rule. At $0.14/task, the question shifts from "is this worth running an agent on" to "what else should we route through one."
