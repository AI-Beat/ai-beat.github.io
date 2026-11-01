---
title: "Five Small Models Beat One Big One"
date: 2026-11-01T06:12:32+00:00
draft: false
slug: five-small-models-beat-one-big-one
categories: [agents]
tags: [agents, coding, benchmark, open-source, qwen]
params:
  author: AI Beat Desk
  summary: >-
    The GVS5H paper showed that five Qwen3.8-27B instances coordinating through a shared file system outperform Fable 5 on LiveCodeBench Hard at roughly 9% of the cost. Memdoor, an open-source harness, independently reproduced the result — 49 of 50 runs solved, $0.39 median per problem.
---

There's a paper from August — [GVS5H](https://arxiv.org/abs/2608.26480), "Zero-Shot Self-Orchestration with Ledger-Based Control for Improved LLM Coding Performance" — that deserves more attention than it got at the time. Its core claim: on the 100 hardest problems in LiveCodeBench, a team of Qwen3.8 Flash Next instances running the GVS5H protocol scored 93.0%, compared to 90.4% for Fable 5. At roughly 9% of the cost. No fine-tuning, no reinforcement learning — just a carefully designed multi-agent harness.

The method is called ledger-based control. Fresh instances of the same model decompose the problem, ideate independently, then hand off to a solver and a reviewer through a shared file system acting as a persistent ledger. Each agent starts with clean context and only sees what the shared files contain, so there's no cross-contamination of reasoning across the parallel branches. The ledger tracks state between agents the way a work queue tracks tasks between workers.

On Halloween, [Memdoor](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve) — an open-source multi-agent workflow runner — published an independent reimplementation. Using the smaller Qwen3.8-27B (not Flash Next) via OpenRouter and the Memdoor workflow engine, they ran five passes on the first ten LiveCodeBench Hard problems. 49 of 50 solved. Median run time 40 minutes, median cost $0.39 per problem. They compare favorably to Fable 5 at 8.0/10 average; the Memdoor implementation averaged 9.8/10.

The caveats matter here. Only ten problems, five passes each — nowhere near the authors' 100-problem evaluation. The contest problems date from March–April 2025, and contamination can't be ruled out (Qwen3.8-27B's training data is opaque on this). The runs used OpenRouter's inference hosts, not the vLLM FP8 setup in the original paper. Two failure modes were documented: one where a manager agent invented a counterexample that led a solver to rewrite a correct solution, and one where a correct program was overwritten in a repair round.

Those failure modes are worth dwelling on. They're not random — they're coordination failures. The agents were communicating correctly through the ledger, but the control flow didn't prevent a downstream agent from accepting a false critique. That's a solvable problem (you can add a verification gate before rewriting anything that already passed tests), but it points at the gap between "the system mostly works" and "the system is reliable." Getting from 49/50 to 50/50, and keeping it there across 100 problems, is where the real engineering is.

Still, the headline result stands. A team of 27B-parameter open-weight models, coordinated through shared files and a structured handoff protocol, is matching the best closed-weight frontier model on hard competitive programming. That's not just a benchmark footnote — it suggests that at the task level, the gap between small-and-coordinated and large-and-solo is narrowing faster than raw parameter counts would imply. The GVS5H gains "were not universal" across the eleven models they tested, but for Qwen3.8 the lift was substantial.

For anyone building coding agent infrastructure, the implementation is worth studying. The Memdoor YAML workflow (planner → parallel ideators → solver → test → review → repair → verify) is short enough to read in a few minutes, and the design principle — agents coordinate through files, not in-context messages — is a clean architectural choice that isolates each agent's reasoning from the others.
