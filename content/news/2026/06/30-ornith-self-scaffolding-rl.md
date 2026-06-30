---
title: "Ornith-1.0: The RL Loop Learns Its Own Harness"
date: 2026-06-30T06:30:00Z
draft: false
slug: ornith-self-scaffolding-rl
categories: [agents]
tags: [agents, open-source, reinforcement-learning, coding, moe]
params:
  author: AI Beat Desk
  summary: >-
    DeepReinforce released Ornith-1.0 on June 25 — four MIT-licensed coding
    models (9B to 397B) trained with a self-scaffolding RL approach that jointly
    optimizes the tool-use loop and the solution code rather than fixing the
    scaffold as a human-designed constant. The 397B variant beats Claude Opus 4.7
    on SWE-Bench Verified and Terminal-Bench 2.1; the 35B MoE beats Qwen 3.5-397B
    on Terminal-Bench at one-eleventh the parameter count.
---

Almost every RL training setup for coding agents treats the scaffold as fixed. You define a tool-use loop — how the agent reads files, runs tests, issues shell commands — and then you optimize the policy that operates within it. The scaffold is a human engineering choice, and a reasonable one: you need a stable environment to get clean reward signals. What Ornith-1.0's authors call "self-scaffolding RL" is an attempt to remove that assumption.

[DeepReinforce](https://ornith.site/) released Ornith-1.0 on June 25 as four MIT-licensed models: 9B Dense, 31B Dense, 35B MoE, and 397B MoE, all built on top of Gemma 4 and Qwen 3.5 base checkpoints. The shared training approach is the distinctive thing. During RL, the model is optimized to produce not just solution code but also the task orchestration framework it uses: the plan, the tool-calling sequence, the intermediate evaluation logic. "Ornith-1.0 treats the scaffold as a learnable object that co-evolves with the model's policy during RL training," per the project description. The reward signal still comes from the final output — tests passing, the problem solved — but the path the model takes to get there, including which tools it invokes and in what order, is something the model learns rather than something a human encodes.

The intuition is straightforward. A fixed scaffold embeds a specific theory of how to solve coding problems: look at the failing tests, hypothesize a fix, implement it, verify. That's a reasonable strategy, but it may not be optimal for every problem type, and as models get stronger the scaffold becomes an increasingly limiting constraint. If the scaffold is itself parameterized and trained, the model can discover more efficient task decompositions — or at least, that's the hypothesis. Whether it actually does this in a principled way or just learns to route around the scaffold's structure in a way that looks effective on benchmarks is a harder question.

The benchmark results are where the argument gets concrete. The 397B MoE variant scores [82.4% on SWE-Bench Verified](https://ornith.site/), against Claude Opus 4.7's 80.8% — a narrow lead, but notable given the MIT license and the relative novelty of the approach. On Terminal-Bench 2.1, which tests more open-ended terminal and shell interactions, Ornith 397B scores 77.5% versus Opus 4.7's 70.3%. That gap is larger and probably more meaningful: terminal tasks require sustained multi-step action with less structured reward signal, which is precisely the regime where scaffold quality matters most.

The smaller models are arguably the more interesting result. The 35B MoE scores 64.2 on Terminal-Bench 2.1, beating Qwen 3.5-397B (53.5) — a model more than ten times larger by total parameter count. That kind of efficiency gap suggests the self-scaffolding training is doing something useful beyond just scaling, though "the small model wins because the scaffold helps" is hard to disentangle from "the small model wins because it was fine-tuned specifically for this benchmark type."

The MIT license and the model sizes make this practically deployable. The 9B Dense variant fits on a single consumer GPU in Q4 quantization; the 35B MoE runs in 25 GB at Q5, which is within reach of a workstation with a couple of good GPUs. The 397B MoE needs around 200 GB in FP8, which puts it in the same tier as running DeepSeek V4 locally — possible but not casual. For anyone building agentic coding infrastructure, the 35B MoE is probably the interesting test case: competitive results at a scale where you can actually iterate.

What remains to be seen is whether the self-scaffolding framing holds up under scrutiny. The benchmark numbers are real, but they do not directly validate the mechanism — a model fine-tuned hard on agentic coding tasks might produce similar results through a more conventional training approach. An ablation comparing Ornith's joint scaffold+solution optimization against a baseline that trains only the solution code with a fixed scaffold would be more convincing. That experiment is not in the public materials yet. In the meantime, the models are available, the license is open, and the results are strong enough to be worth evaluating against whatever agentic coding setup you are currently using.
