---
title: "The Formatting Tax on Reasoning Models"
date: 2026-05-24T06:09:37Z
draft: false
slug: delta-token-credit-rlvr
categories: [research]
tags: [rlvr, training, reasoning, gradients, reinforcement-learning]
params:
  author: AI Beat Desk
  summary: >-
    DelTA identifies a structural problem in RLVR training: the gradient signal used
    to improve reasoning models is dominated by high-frequency formatting tokens rather
    than the tokens that actually distinguish good responses from bad ones. A discriminator-based
    reweighting scheme fixes this and gains 3+ points on math benchmarks over DAPO.
---

Reinforcement learning from verifiable rewards has become the standard recipe for improving reasoning in large language models. The basic loop is straightforward: generate a batch of responses to math or coding problems, check them against a verifier, assign a reward of 1 or 0, and use the reward signal to update the model. The tokens in high-reward responses get their probabilities pushed up; those in low-reward responses get pushed down. It works — the improvements on math benchmarks have been substantial — but a new paper called [DelTA](https://arxiv.org/abs/2605.21467) points out that it doesn't quite work as intended.

The problem starts with what actually varies between a correct and incorrect response. The setup prompt is identical. The response format is usually similar: chain-of-thought scaffolding, closing phrases like "Therefore," code fencing like ```` ``` ````, whitespace. These structural tokens appear in both positive and negative samples in roughly similar proportions. When you compute the gradient signal by subtracting the negative-side centroid from the positive-side centroid, these shared tokens contribute noise rather than signal — they push in both directions and partially cancel, but also dominate by sheer frequency. The tokens that actually discriminate — the specific algebraic moves or logical transitions that separate correct reasoning from incorrect — get diluted.

DelTA frames this as a discrimination problem and builds a solution from that framing. The update direction in RLVR can be understood as a linear discriminator over token-gradient vectors: the question is which tokens' gradient directions most reliably separate the positive class from the negative class. Standard RLVR builds the discriminator naively, from advantage-weighted averages. DelTA instead runs an iterative refinement: estimate which tokens are more characteristic of their own advantage side, recompute the centroids with those tokens upweighted, repeat. The resulting per-token coefficients are used to reweight the RLVR surrogate objective.

The implementation is light. The coefficient estimates use layer-restricted LM-head gradients rather than full-parameter gradients — a computational compromise — and are bounded to \([0.8, 1.2]\) to prevent extreme rescaling. One refinement iteration is sufficient in practice. These choices matter for adoption: a training modification that doubles compute overhead doesn't get used; one that adds a modest bookkeeping step does.

The results are on Qwen3-8B-Base and Qwen3-14B-Base trained on DeepMath-103K, evaluated on seven math benchmarks. DelTA over DAPO: +3.26 average points at 8B, +2.62 at 14B. The improvements hold consistently across AIME, HMMT, and Brumo, not just one benchmark that happened to align with the training distribution. Code generation generalizes similarly. A different backbone (Olmo3-7B-Base) shows the same pattern.

The ablation that makes the clearest case: rank tokens by their DelTA coefficient. Train on only the top-ranked tokens. Accuracy improves. Train on only the bottom-ranked tokens. Accuracy falls below the baseline. The method is identifying tokens that are genuinely causally involved in the response quality difference, not just correlates.

There is an honest limitation buried in the paper. The gradient proxy (restricted to the LM head) is an approximation to full-parameter gradients. Whether the approximation holds at larger scales or with different architectures isn't established. The evaluation is also heavily weighted toward mathematical reasoning — the code generation results are promising but less detailed.

What DelTA makes concrete is something that practitioners have probably intuited: that RLVR training is noisier than it looks, and that the noise isn't uniform. Responses from good and bad trajectories look similar at the token level for most of their length. The signal is sparse and concentrated in particular moments. A method that can identify those moments and weight them appropriately is doing something the base RLVR setup cannot.

The gain of 3 points may seem modest next to the headline improvements of frontier model releases, but at the level of training methodology, it's meaningful. These gains compound: a better RLVR signal means every training step does more useful work, which means you get more reasoning capability per compute dollar spent. That matters as the field moves toward training on progressively harder problems where correct responses are rare and the signal extraction problem only gets worse.
