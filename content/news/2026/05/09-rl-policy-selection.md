---
title: "RL Doesn't Teach Reasoning. It Picks a Lane."
date: 2026-05-09T06:12:02+00:00
draft: false
slug: rl-policy-selection-not-capability-expansion
categories: [research]
tags: [reasoning, rl, training, paper]
params:
  author: AI Beat Desk
  summary: >-
    A new paper argues that reinforcement learning on reasoning tasks doesn't
    teach models new problem-solving strategies — it redistributes probability
    mass over solutions the base model already contains. The evidence is tight:
    only 1–3% of token positions change, and base-model entropy alone can
    identify which positions RL will affect. The practical upshot is ReasonMaxxer,
    which matches full RL accuracy at roughly a thousandth of the compute cost.
---

The dominant story about how frontier reasoning models got good at math goes something like this: you take a base LLM, apply reinforcement learning on verifiable tasks (code execution, math answer-checking), and the model *learns* new reasoning strategies. The chain-of-thought, the backtracking, the step-checking — all of it emerges from RL. Models like o1 and R1 are regularly described as having acquired capabilities their base counterparts simply don't have.

A new paper, ["Rethinking RL for LLM Reasoning"](https://arxiv.org/abs/2605.06241) by Akgül and colleagues, challenges this picture directly. Their argument: RL doesn't expand what a model can do. It selects from what the model already knows how to do.

## The Claim and the Evidence

The core finding is empirical, not theoretical. When the authors examine exactly where RL changes a model's outputs, they find that only 1–3% of token positions differ between RL-trained and base model outputs. More importantly, those positions aren't random: the promoted token at each affected position is always within the base model's top-5 alternatives. RL isn't writing new tokens into the model's vocabulary of solutions; it's upweighting ones that were already plausible.

The causal check is clever. If you go into a base model's output, find the positions RL would affect (identifiable from the base model's own entropy — no RL model required), and manually push the token toward what RL would have chosen, you recover a large fraction of RL's accuracy gain. Random corrections at other positions don't help. This strongly suggests that what RL provides is a *selection mechanism* over existing strategies rather than new strategies.

They also show that the accuracy improvement is representable in a tiny fraction of model parameters — further evidence that the change is shallow in a meaningful sense.

## What ReasonMaxxer Does with This

If the value of RL is in identifying and upweighting certain token positions, you don't necessarily need the full RL training loop to capture that value. The authors build ReasonMaxxer around this insight: use the base model's entropy to identify decision-critical positions, apply targeted interventions at those positions (via a small auxiliary model trained cheaply), and skip the expensive RL loop. The result matches full RL accuracy at roughly three orders of magnitude lower compute cost.

To be clear about the scope: this is tested on the kind of benchmarks RL reasoning is usually evaluated against (math, logical reasoning). It's not a claim about every use of RL in training, and it doesn't address RLHF for value alignment, which operates differently.

## Why This Matters More Than a Negative Result

The instinctive reaction might be: if RL just selects from existing capabilities, that's disappointing — it means you can't keep improving by running more RL once the base model's capability ceiling is hit. But the more interesting implication runs the other direction.

If the gains from RL for reasoning are concentrated in a tiny slice of tokens that the base model already has high uncertainty about, then what the base model knows — and how it distributes probability over alternatives at those positions — is the real variable. The practical ceiling for RL-based reasoning improvement is then set by base model pretraining, not by the RL procedure. Getting a better model isn't primarily about longer RL runs; it's about a better base.

This realigns where you'd want to invest compute if your goal is higher reasoning accuracy. It also suggests that the architectural and data choices made during pretraining have more downstream influence on reasoning performance than the RL stage gets credit for.

There's also a crisper version of a nagging concern about reasoning evaluation: if RL is selecting from existing capabilities rather than generating new ones, benchmarks where the base model already has the answer somewhere in its distribution but rarely surfaces it may dramatically overstate the gap between RL-trained and non-RL-trained models on *genuinely novel* problems. Whether that concern is well-founded depends on how well current benchmarks actually probe novel reasoning — a question this paper leaves open.

The practical import, though, is clear: if ReasonMaxxer holds up on a broader range of tasks, the cost of matching frontier reasoning accuracy just dropped significantly for anyone doing this at smaller scale.
