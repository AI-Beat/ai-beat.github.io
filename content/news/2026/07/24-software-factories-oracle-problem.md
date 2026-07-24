---
title: "The Oracle Problem That Lights-Off Software Factories Can't Solve"
date: 2026-07-24T08:12:30+02:00
draft: false
slug: software-factories-oracle-problem
categories: [agents]
tags: [agents, coding, software, rl, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    HumanLayer's essay "Why Software Factories Fail" makes a focused argument:
    the ceiling on autonomous coding isn't harness engineering but the absence
    of a fast oracle for architectural quality. RL can't reward maintainability
    because tests take seconds and design debt takes months. The fix isn't more
    scaffolding — it's restructuring where humans stay in the loop.
---

The phrase "software factory" has been circulating for about eighteen months — fully autonomous pipelines where AI agents write, test, and ship code while humans watch dashboards. The [HumanLayer team](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md) published a short essay today called "Why Software Factories Fail (or: harness engineering is not enough)" and it makes an argument worth reading carefully, not least because they sell human-in-the-loop tooling and have an obvious reason to want the lights-off dream to fail.

Their argument isn't that autonomous coding doesn't work. It's that it degrades. Models excel at solving isolated problems quickly — a test passes, a bug closes, a feature ships. But they systematically erode maintainability through poor design decisions: inconsistent abstractions, duplicated logic, brittle coupling. The individual tasks look fine; the codebase quietly becomes harder to change.

The diagnosis is about feedback loops. Reinforcement learning trains on what can be measured fast. Tests run in seconds; a binary pass/fail signal is easy to compute and reward. But good architecture is only visible over weeks or months of subsequent development. There's no fast oracle that reliably answers "is this the right abstraction?" or "will this make the codebase harder to extend?" That means RL can't learn to prefer maintainable designs over expedient ones, because the penalty for bad design arrives too late — if it arrives at all.

This isn't just a harness problem, which is the essay's central claim. The field has spent the past year building ever more sophisticated agent harnesses: better linter integrations, richer context injection, tighter test loops, code review agents that read diffs. [Ryan Lopopolo's work on harness engineering](https://www.augmentcode.com/guides/harness-engineering-ai-coding-agents) is cited as the intellectual center of this approach. The HumanLayer argument is that no amount of environmental scaffolding can compensate for a model that hasn't been trained to care about the properties scaffolding can't measure.

There's a useful analogy here to program synthesis research. The verifier problem in formal methods is well-known: if you can fully specify correctness, you can generate and check candidates automatically. Software factories work when "correctness" means passing tests. They break when correctness means maintaining a coherent design that humans can reason about, extend, and debug — a property that resists specification precisely because it depends on future, unknowable contexts.

The practical prescription is to restructure human involvement rather than eliminate it. Instead of reviewing code after agents write it (expensive, slow, often after the architectural damage is done), front-load the human decisions that shape what agents will build: product requirements, system architecture, program design, and vertical slices of the target functionality. Let agents do the implementation given tight, human-specified constraints. The claimed result is 2–3× speed improvement with sustainable code quality — not a lights-off factory, but a competent co-production mode.

The essay's honest limitation is also worth naming: HumanLayer is the company that sells human-in-the-loop infrastructure for AI agents. They have a financial interest in the conclusion. The argument stands or falls on its merits, not the messenger, but the structural conflict is real.

What makes the argument compelling despite that conflict is specificity. The oracle problem — no fast reward signal for architectural quality — isn't just asserted; it's traceable to how current RL training works. SWE-bench uses pass/fail on existing tests. There's no scoring dimension for "the solution doesn't make the next ten tasks harder." That's a training gap, and it's not obvious how to close it without either a dramatic improvement in static analysis tooling or a much longer feedback horizon than any current training setup supports.

The lights-off factory will probably remain a useful framing for narrow, well-specified workflows: data pipeline maintenance, localization, certain classes of dependency updates. For anything that involves ongoing design judgment, the essay's forecast — that harness engineering alone won't get you there — seems more right than the confident predictions that claim otherwise.
