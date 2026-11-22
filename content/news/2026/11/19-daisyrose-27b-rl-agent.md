---
title: "1,200 Problems, Carefully Chosen: DaisyRose-27B"
date: 2026-11-19T06:30:00Z
draft: false
slug: daisyrose-27b-rl-agent
categories: [training]
tags: [reinforcement-learning, agents, open-source, fine-tuning]
params:
  author: AI Beat Desk
  summary: >-
    RadixArk's DaisyRose-27B fine-tunes Qwen3.8-27B with RL over ~1,200 curated
    long-horizon agent tasks, reaching 12.0% pass@1 on Terminal-Bench 4.1 from a
    5.6% base. The interesting part isn't the benchmark gain but how they picked
    the data: each task targets a specific error the base model makes and is
    tuned to a moderate solve rate to stay in the RL learning zone.
---

[RadixArk](https://www.radixark.com/blog/daisyrose-27b) released DaisyRose-27B yesterday — a fine-tuned version of Qwen3.8-27B trained with reinforcement learning over long-horizon agent tasks. The headline number is a Terminal-Bench 4.1 pass@1 improvement from 5.6% to 12.0%, which the company describes as more than double on "a notoriously hard long-horizon benchmark." The underlying story is worth reading closely, because the approach says something useful about where RL fine-tuning for agents is currently at.

The training set contains approximately 1,200 tasks. That's small. For context, typical SFT datasets for coding assistants run to hundreds of thousands of examples. The data was curated with Eigent AI, and the curation criteria were specific: each task had to target an error class the base model actually makes, had to be calibrated to a moderate solve rate (not too easy, not impossible), and had to be reviewed for reward hacking — cases where the model learns to satisfy the reward signal without solving the actual problem. The filtering is deliberate because RL for long-horizon tasks is brittle; if the problem is too easy, the model learns nothing; if too hard, the gradient signal is noise; if hackable, the model learns the wrong thing loudly.

Each training session ran up to 1 million tokens. That's the other dimension of "long-horizon" — these aren't tasks where you write a function and check if it passes unit tests. They're the kind where the agent has to plan, execute, check, fail, revise, and eventually succeed over a session that might span dozens of tool calls. Getting useful gradient signal out of sessions that long requires both good tasks and an infrastructure that can actually run them at training scale.

RadixArk built that infrastructure around fully async training: 128 H200 GPUs running agent inference in parallel, 16 running the trainer. Training took 18 days and 944,000 sandbox executions. They report 57% lower training cost and roughly 2× faster throughput compared to what they call "the open-source trainer" — presumably referring to common setups built on GRPO/PPO pipelines. The sandbox environment is based on [AgentENV](https://www.radixark.com/blog/daisyrose-27b), which they claim costs about 80% less per sandbox run than mainstream cloud sandbox providers.

The benchmarks beyond Terminal-Bench 4.1 — DeepSWE, SWE-Bench Pro, HLE, IFBench, GPQA Diamond — all show improvement over the base model, though the numbers aren't broken out in detail in the post. Terminal-Bench 4.1 is the clearest signal because it's explicitly designed to resist the kinds of shortcuts that inflate easier agent benchmarks.

A few things this doesn't answer: how DaisyRose-27B compares against other RL-trained agent models at similar scale (there's no head-to-head), and whether the Terminal-Bench gains transfer to realistic deployment environments outside of standardized evals. The weights are on Hugging Face; the license isn't stated in the blog post, which is an odd omission worth checking before building on it.

What the release does contribute is a reasonably well-documented case study in data-efficient RL fine-tuning. The argument implicit in the approach is that the selection function over training tasks matters as much as compute — maybe more. At a time when many labs default to scaling training data and hoping for coverage, 1,200 carefully chosen problems producing a 2× benchmark improvement is worth taking seriously.
