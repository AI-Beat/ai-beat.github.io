---
title: "The Text-Space Optimizer"
date: 2026-05-27T06:13:00Z
draft: false
slug: skillopt-text-space-optimizer-agent-skills
categories: [research]
tags: [agents, skill-optimization, research, rl]
params:
  author: AI Beat Desk
  summary: >-
    SkillOpt treats agent skill optimization as gradient descent in text space:
    a separate optimizer model proposes bounded edits to skill documents, commits
    only what strictly improves validation performance, and uses a rejected-edit
    buffer as a form of momentum. Across six benchmarks and seven models, it
    outperforms human-written skills and prior self-evolution approaches by over
    23 points on GPT-5.5 in coding environments.
---

Agent skills — the instruction documents that tell an agent how to use a specific tool, follow conventions, or handle a domain — are usually written once and updated rarely. When they do get updated, it tends to look like: pass the model its current skill file, ask it to suggest improvements, accept or reject based on intuition. Repeat occasionally.

[SkillOpt](https://arxiv.org/abs/2605.23904), a paper out of May 22, makes the case that this is the equivalent of training a neural network by manually adjusting weights based on feel: it works sometimes, scales poorly, and gives you no systematic way to know if you're actually improving or just polishing prose. The paper proposes treating skill improvement as a controlled optimization problem, applying the structure of gradient descent to text.

## The optimization loop

A separate optimizer model — not the agent itself — reviews scored rollouts from the current skill document and proposes bounded edits: add a sentence, delete a confusing instruction, replace a vague phrase. The edits are constrained in scope, which matters. Unconstrained rewriting can change everything at once, making it impossible to attribute improvements or regressions to specific changes. Bounded edits let you track what changed and why it helped.

The proposed edit gets tested against a held-out validation set. If performance improves, the edit is committed to the skill document. If it doesn't, the edit is rejected — and stored in a buffer. That buffer is the key mechanism: it informs future proposals, preventing the optimizer from cycling back through the same bad ideas. It's functionally similar to momentum in gradient descent, using past failure information to shape the search direction.

The system also includes an epoch-wise meta-update step that adjusts the optimizer's proposal strategy based on recent history, and a "textual learning rate" that bounds how aggressively any single edit can change the skill. The analogy to gradient descent is explicit in the paper and holds up reasonably well: textual learning rate corresponds to step size, the rejected-edit buffer corresponds to momentum, validation performance is the loss, and the optimizer analyzing rollouts corresponds to computing gradients.

## Results

Across six evaluation benchmarks and seven models, SkillOpt is best or tied across all 52 evaluated (model, benchmark, harness) combinations — outperforming human-written skills, one-shot LLM generation, TextGrad, GEPA, Trace2Skill, and EvoSkill. On GPT-5.5, accuracy gains exceeded 23 points in direct chat settings and approached 25 points in agentic coding environments. Skills also transferred: a skill optimized on one model produced meaningful gains when deployed with a different model, suggesting the optimization is learning something generalizable about the task rather than overfitting to model-specific quirks.

Adding inference-time overhead is conspicuously absent: the optimization runs offline during skill development, not at query time. The optimized skill document is just a better text file. Runtime cost is the same as before.

## The broader point

The paper's real argument is about what "self-improvement" means at the skill level. Asking a model to revise its own instructions tends to produce local polishing: clearer language, more examples, reorganized sections. The optimization loop in SkillOpt instead targets measurable task performance, so edits are selected for effect rather than readability. That distinction matters as agents take on longer-horizon tasks where skill quality is directly load-bearing — not just a stylistic choice but the difference between completing the task and failing it.

The question the paper leaves open is how this scales when skills interact. A single skill document is a clean optimization target. Agents that chain multiple skills, or that dynamically compose instructions, introduce dependencies that make isolated optimization less reliable. That's a reasonable problem to leave for follow-up. The baseline case — optimizing a skill in isolation for a defined task — is cleaner and the gains are real.
