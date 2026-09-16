---
title: "RL Training Has a Hard Problem Problem"
date: 2026-09-16T06:30:00+00:00
draft: false
slug: ngu-rl-hard-problems
categories: [training]
tags: [training, reinforcement-learning, rl, grpo, math, code]
params:
  author: AI Beat Desk
  summary: >-
    A new paper and blog post from Michael Noukhovitch identifies the "Matthew
    Effect" in RL training for LLMs: standard GRPO concentrates gains on
    problems a model can already mostly solve, while genuinely hard problems
    see minimal improvement. The fix — Never Give Up (NGU) — is an adaptive
    sampling strategy that keeps generating completions until it finds one that
    works, naturally redistributing compute toward the problems that need it.
---

There's a structural problem with how reinforcement learning training for LLMs works, and it's been hiding in plain sight. Michael Noukhovitch's [new paper and blog post](https://mnoukhov.github.io/posts/ngu/) — arxiv [2609.13443](https://arxiv.org/abs/2609.13443) — gives it a name: the Matthew Effect. "For he who has much, more will be given."

The mechanics are straightforward. Standard GRPO (Group Relative Policy Optimization) draws a fixed batch of prompts and generates a fixed number of completions \(k\) per prompt, then updates on whichever got the right answer. The problem: if a model already solves a problem 70% of the time, it will reliably find a correct completion in a batch of \(k=8\) outputs and get a gradient update. If it solves a problem 2% of the time — the hard cases — the odds that any of its \(k=8\) completions is correct drop to roughly 15%. No correct completion, no useful signal. Those problems either starve the training signal or, if you're unlucky, get updates from noise.

The result is that RL training makes models better at problems they were already decent at, while the genuinely hard tail barely moves. This shows up empirically on math benchmarks: models post large RL gains on AIME problems they could sometimes solve, while the hardest AIME subsets remain essentially untouched. It's not that RL doesn't work — it's that it's spending its compute budget where returns are already diminishing.

The proposed fix, Never Give Up (NGU), is satisfyingly simple. Instead of sampling \(k\) completions and moving on, the algorithm keeps sampling until it finds a correct one, then trains on everything it accumulated. Easy problems get filtered out quickly — the model already solves them in a small number of tries, so the overhead is low. Hard problems get whatever compute they need. The name comes from the core loop: don't give up on a problem until you have at least one correct example to learn from.

In practice the sampling is asynchronous, which matters for throughput: the system doesn't block waiting on hard prompts. Easy and medium problems stream through normally; only the long tail accumulates large batches of completions before producing a training example. On the Deepscaler math benchmark and Manufactoria coding tasks (a tricky domain that requires building logic gates from a constrained set of primitives), NGU outperforms vanilla GRPO especially on the hardest subsets, while maintaining similar performance on easy problems.

The paper also examines the off-policy question carefully: when you're accumulating many completions before producing an update, the policy used to generate early completions differs from the policy at update time. The authors find that in practice this doesn't cause instability on the tasks they test, and they discuss why — the cumulative distribution of correct completions doesn't shift dramatically between early and later samples, so the gradient estimates remain reasonable. More empirical validation on diverse tasks would be useful, but the off-policy concern seems less severe in practice than in theory.

What makes this interesting beyond the immediate result is what it implies about RL training design more broadly. Most RL-for-LLMs work optimizes around the training pipeline — better reward shaping, better advantage estimation, curriculum ordering. NGU is attacking the sampling layer: the question of which problems should actually produce gradients and how many shots each problem should get. The insight that compute reallocation is a first-class design choice, not just an afterthought, seems right and probably has more applications than just this particular algorithm.

The code is [available alongside the blog post](https://mnoukhov.github.io/posts/ngu/), and the approach is model-agnostic — the authors run experiments with Qwen models, which are widely accessible, so it should be reproducible.
