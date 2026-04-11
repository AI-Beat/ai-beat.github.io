---
title: "AI Wins Live Codeforces Rounds, Three in a Row"
date: 2026-04-06T06:20:00+00:00
draft: false
slug: grandcode-codeforces-live
tags: [agents, reinforcement-learning, competitive-programming, coding]
categories: [agents]
params:
  author: AI Beat Desk
  summary: >-
    A preprint from the DeepReinforce Team claims their GrandCode system placed
    first in three consecutive live Codeforces rounds in March 2026, defeating
    all human participants. The technical contribution is Agentic GRPO, a
    multi-stage RL algorithm designed for agent pipelines where reward signals
    arrive late and off-policy drift is severe. Take the claim seriously, but
    verify the details before the hype cycle arrives.
---

A preprint posted to arXiv on April 3 describes [GrandCode](https://arxiv.org/abs/2604.02721), a system from the DeepReinforce Team that claims to have placed first in three consecutive live [Codeforces](https://codeforces.com) rounds — rounds 1087, 1088, and 1089 — in late March 2026, beating all human participants including what the paper calls "legendary grandmasters."

That's a significant claim if it holds up. For context, the best prior AI result on competitive programming was Google's Gemini 3 Deep Think, which reached 8th place — and that was under controlled, not live, conditions. Coming first across three consecutive live contests would represent a qualitative shift: not "approaching human expert performance" but "currently the best thing competing in this environment."

The technical core is a multi-agent orchestration system with specialized modules: hypothesis proposal, solver, test generation, and summarization. These agents are jointly trained using a new algorithm the team calls Agentic GRPO, which adapts Group Relative Policy Optimization (GRPO) for multi-stage agent rollouts. GRPO was designed for single-step preference optimization; its challenges when applied to pipelines include delayed rewards (the final test verdict on a competitive programming problem arrives long after the first hypothesis was generated) and severe off-policy drift (as the policy updates, the rollouts it generated earlier become increasingly unrepresentative, making credit assignment noisy).

The paper's specific contribution is addressing these two failure modes — delayed reward attribution across a pipeline and off-policy drift when the agent's behavior diverges from the collected training data. Whether the proposed solutions are principled or just pragmatic hacks that happened to work on this benchmark isn't clear from the abstract alone.

There are real caveats. This is a preprint, not a peer-reviewed result. The abstract discloses no model size, no base model, no training compute, and no ablations. The three Codeforces rounds were in March 2026, and the paper was posted days later — which leaves little time for independent verification or replication. There's no institutional affiliation listed. The paper cites only one comparison point (Gemini 3's 8th place), and that comparison isn't even apples-to-apples since the conditions differ.

None of that makes the result implausible. Competitive programming has historically been a good benchmark precisely because it's hard to overfit a live contest — the problems are novel and timing is real. If the Codeforces community can confirm the contest accounts and submissions, the underlying claim becomes much more credible.

What's interesting regardless of whether the specific live-contest claim fully checks out is the framing: agentic RL at test time, across a multi-module pipeline, with coordinated training. This is the same pattern appearing across coding, math, and browser-control research right now — not a single model doing chain-of-thought, but a structured collection of specialized agents that propose, solve, verify, and summarize, optimized end-to-end with RL. The difference from classical symbolic AI approaches is that here there's no explicit hand-coded search or rule system; the agents learn their coordination through post-training.

Whether "consistent grandmaster level" means first in every contest going forward, or three specific rounds where conditions were favorable, matters a lot. But the direction of travel is clear. These systems are no longer merely useful programming assistants — they're becoming competitive participants in environments designed to separate the best human problem solvers from the rest.

The GrandCode code and model weights were not released with the preprint.
