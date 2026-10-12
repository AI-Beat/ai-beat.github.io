---
title: "A Model That Teaches Itself to Coordinate"
date: 2026-10-12T06:12:00+00:00
draft: false
slug: mass-recursive-self-improvement
categories: [training]
tags: [self-improvement, multi-agent, fine-tuning, LoRA, Sakana]
params:
  author: AI Beat Desk
  summary: >-
    Sakana AI and UC Berkeley's MASS paper frames recursive self-improvement not
    as a single model thinking harder, but as a cycle of evolutionary workflow
    search, self-evaluation, and LoRA fine-tuning on the model's own best
    multi-agent traces — with the workflow text deliberately withheld during
    training.
---

One recurring problem in alignment and capability research is that the more capable a model becomes, the harder it is to supervise. You can't use human judgment to grade every output at scale, and synthetic tasks with ground-truth answers cover only a narrow slice of what models are actually deployed to do. [MASS](https://arxiv.org/abs/2610.12176), a paper from UC Berkeley and Sakana AI posted to arXiv this week, takes a specific stab at this: recursive self-improvement on open-ended, non-verifiable tasks.

The core idea is that a single base model takes on three roles — task solver, workflow optimizer, and evaluator — and runs an alternating loop. In the inner loop, the model generates candidate multi-agent workflows, runs them, pairwise-judges the outputs, and revises the workflow accordingly. In the outer loop, it runs the best workflow many times with different seeds, ranks the resulting trajectories, and fine-tunes itself with LoRA on the top outputs.

The twist that makes this technically interesting rather than tautological: when training on those winning trajectories, the workflow specification is stripped from the prompts. The model sees only the task and the execution trace. It learns how to coordinate without being handed the coordination script. The bet is that the model internalizes something about effective multi-agent interaction rather than just memorizing a particular workflow template.

Starting from Qwen3.6-27B and running two MASS cycles, the win rate against the base model on held-out test tasks rose from 53.9% to 69.9%, judged by external models. On public benchmarks — MLR-Bench, DSBench, ScienceAgentBench, AstaBench — performance per output token came out 1.2–1.6× the base model. Self-evaluation agreement with external judges improved from 73% to 93%.

There's also an ablation result worth noting. When the authors upgraded only the evaluator (better grading, same optimizer), gains were modest. Upgrading the optimizer as well made the bigger difference. This suggests the bottleneck in these workflows isn't knowing which output is good — it's knowing how to structure the attempt in the first place.

Transfer was weaker on coding benchmarks like SWE-bench Verified and Terminal-Bench 2.0, where scores sat at roughly 0.94–0.96× the base. The authors frame this as a domain specificity issue; the workflows optimized on research tasks didn't cleanly transfer to software engineering tasks. That's an honest limitation — and it means the approach probably needs task-specific cycles rather than a single general-purpose RSI run.

The multi-agent trajectory angle has an interesting training data implication that the paper develops: a multi-agent trace packs more supervision signal per token than a single-agent trace of the same length, because you get multiple perspectives on the same task in a single sequence. Whether this advantage scales is an open question, but it suggests that deliberately structuring training data as agent dialogues rather than monologues might be worth exploring outside the RSI context.

What MASS doesn't do is improve the model's weights in a way that generalizes beyond the task distribution it was trained on, at least not dramatically. Two cycles of self-improvement on research benchmarks gave a meaningful lift, but the gains are benchmark-adjacent. Whether the coordination patterns learned here transfer to genuinely novel situations remains to be seen. Still, the paper is a careful, reproducible proof of concept that a model can supervise its own improvement on tasks where humans can't easily grade the outputs — which is a non-trivial thing to demonstrate.
