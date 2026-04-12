---
title: "The Moat Is the System, Not the Model"
date: 2026-04-12T06:14:29Z
draft: false
slug: cybersecurity-jagged-frontier
categories: [research]
tags: [security, models, capability]
params:
  author: AI Beat Desk
  summary: >-
    AISLE tested Anthropic's Mythos cybersecurity showcase cases against eight
    open-weight models from 3.6B to 120B parameters. All eight reproduced the
    FreeBSD NFS exploit. A 5.1B model traced the OpenBSD integer overflow chain.
    Smaller open models beat frontier labs on false-positive detection.
    Capability in this domain doesn't scale smoothly — the system architecture
    matters more than raw model size.
---

When Anthropic shared preview results from Claude Mythos last week, the cybersecurity findings were framed as evidence of frontier-exclusive capability: a system that discovered a novel vulnerability in FreeBSD's NFS stack and traced the integer overflow chain in a 27-year-old OpenBSD bug. The implication — at least in the way it was communicated — was that this kind of work required scale.

AISLE's [replication study](https://aisle.com/blog/ai-cybersecurity-after-mythos-the-jagged-frontier), published April 7, complicates that narrative.

The researchers tested the same three vulnerability cases — FreeBSD NFS exploit detection, OpenBSD SACK integer overflow analysis, and OWASP false-positive detection — against eight models spanning a wide capability range. The smallest was a 3.6B-parameter model costing $0.11 per million tokens. All eight models identified the FreeBSD NFS vulnerability. A 5.1B open model "recovered the core chain" of the OpenBSD SACK bug, reproducing the mathematical reasoning about signed integer overflow that Anthropic had highlighted as a flagship result. On the OWASP false-positive test — data-flow tracing, arguably the hardest of the three cases — smaller open models outperformed models from the major frontier labs.

The framing AISLE offers is that "cybersecurity capability is very jagged: it doesn't scale smoothly with model size." This fits what practitioners working with LLMs on security tasks have been observing informally for a while. Vulnerability analysis is a domain where the relevant knowledge is relatively discrete — you either know the FreeBSD NFS code paths or you don't, you either understand signed integer overflow semantics or you don't — and smaller models trained on the right data can close most of the gap with much larger general-purpose models. This is different from tasks where generalization across novel contexts really does require scale.

The paper's conclusion — "the moat in AI cybersecurity is the system, not the model" — points toward orchestration scaffolding, iterative validation loops, and access to relevant codebases as the real differentiators. That's reassuring in one sense (you don't need exclusive access to a trillion-parameter model to do useful security research) and clarifying in another (the competitive moat labs might want to claim for frontier-only cybersecurity capabilities is narrower than the Mythos framing suggested).

There's an important caveat here. AISLE tested the specific cases Anthropic highlighted as flagship results — this was the right thing to do, because those cases are what the capability claim was based on. But it doesn't cover the full range of what Mythos can do, and it's entirely possible that qualitatively harder tasks would show a larger gap. Replicating the showcase examples doesn't mean small models are equivalent overall. It means those particular examples were replicable.

What it does mean is that security teams evaluating whether to deploy AI assistance don't need to wait for access to the largest available models. The bottleneck is more likely to be tooling — how you orchestrate model calls, how you iterate on candidate vulnerabilities, how you integrate with build systems and CI pipelines — than raw model capability. That's a useful recalibration for teams trying to decide where to invest.
