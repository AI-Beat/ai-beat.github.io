---
title: "Training Without Gradients"
date: 2026-10-06T06:15:00+00:00
draft: false
slug: dust-backprop-free-training
categories: [research]
tags: [training, optimization, transformers]
params:
  author: AI Beat Desk
  summary: >-
    Q Labs Research's Dust drops backpropagation entirely and replaces it with
    activation-space perturbation, where each token acts as a virtual population
    member in a zeroth-order optimizer. The approach is orders of magnitude more
    efficient than prior evolution-strategy baselines and closes much of the gap
    with backprop at scale.
---

Backpropagation has been the default training algorithm for neural networks for decades. Not because it's the only option — evolution strategies, forward-mode differentiation, and local learning rules have all been proposed — but because it's fast, accurate, and the hardware ecosystem was built around it. You can get first-order gradient estimates for every parameter in a single forward-backward pass, and GPUs are very good at the backward pass.

Q Labs Research published [Dust](https://qlabs.sh/research/dust) this month as a zeroth-order alternative that trains transformers using only forward passes. The core idea is elegant enough to explain in a paragraph.

## The virtual population trick

Standard evolution strategies perturb the model's *weights*, which means you need multiple copies of the model to estimate a gradient — expensive both in memory and compute. Dust perturbs *activations* instead, and exploits the sequence dimension to run the entire virtual population in a single forward pass. Each token position gets an independent Gaussian noise vector added to its activations. Because each token is processed in parallel, you effectively evaluate thousands of noisy variants of the network simultaneously without replicating the weights.

Credit assignment works by computing how much each perturbation reduced the loss at the current token and at future tokens (with exponential decay across the sequence). The reward-weighted average of the noise vectors gives an estimate of the gradient. This is the REINFORCE estimator applied to activation noise rather than policy parameters — the math is not new, but the key move of treating token positions as population members is.

The gradient estimates this produces are noisy, but they're not arbitrary: the paper reports that cosine similarity with backpropagation gradients increases predictably as population size (token count × draws) grows, and at 16K draws, Dust's estimates closely track what backprop would produce.

## What the benchmarks actually show

At small token budgets (under ~10M tokens), backprop still wins by a significant margin. That's expected — first-order methods accumulate signal faster than zeroth-order ones. But as token budget grows, the gap narrows, and at 20M tokens with sufficient population size, Dust matches or slightly exceeds backprop's validation loss on 10M and 20M token comparisons.

The efficiency gain over prior evolution-strategy baselines is more striking. Dust outperforms EGGROLL — a recent weight-space ES baseline — by \(10^3\) to \(10^4\)× at comparable population sizes. The difference comes almost entirely from the virtual-population trick: instead of running N separate forward passes to get N function evaluations, you run one pass with N token positions and get N evaluations for free.

There's also a scaling result worth noting: larger models are *more* population-efficient, not less. A 243M parameter transformer requires fewer samples to achieve a given gradient quality than a 1.8M parameter one. The intuition is that larger models have smoother loss landscapes relative to their expressivity, so activation perturbations are more informative. This is preliminary evidence, but it suggests the approach doesn't hit a wall as models get bigger.

## Why this matters and where it doesn't

The most obvious practical application is on-device fine-tuning. Backpropagation requires storing activations for every layer during the forward pass to use during the backward pass — this roughly doubles peak memory during training. Dust requires no activation storage. On a device where inference memory is already at the limit, that's the difference between fine-tuning being feasible and not.

The deeper implication is more speculative. Backpropagation requires a differentiable computation graph, which limits what you can include in training: discrete sampling steps, non-differentiable oracles, external tools. Dust is indifferent to differentiability — you can inject any black-box perturbation and measure its effect on the loss. Whether the gradient estimates stay useful as the computation becomes more structured is an open question the paper doesn't address.

The code is at [github.com/qlabs-eng/dust](https://github.com/qlabs-eng/dust). The authors are from Q Labs Research, which appears to be a small independent lab — this isn't a large team with a lot of compute, which makes the result more interesting rather than less.
