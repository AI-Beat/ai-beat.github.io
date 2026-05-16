---
title: "The Draft Model You Don't Have to Train"
date: 2026-05-16T06:05:48+00:00
draft: false
slug: orthrus-dual-view-diffusion-decoding
categories: [research]
tags: [inference, speculative-decoding, diffusion, efficiency, lossless, qwen3]
params:
  author: AI Beat Desk
  summary: >-
    Orthrus (arXiv 2605.12825) grafts a trainable diffusion head onto a frozen
    AR backbone, sharing the exact same KV cache. An intra-model consensus
    mechanism guarantees that every accepted token matches the AR distribution
    exactly — no approximation, no quality tradeoff — while achieving up to 7.8×
    speedup on Qwen3-8B with only O(1) memory overhead. The approach sidesteps
    the core operational cost of speculative decoding: maintaining a separate,
    carefully calibrated draft model.
---

Speculative decoding has become the default inference trick in production LLM serving. The recipe is well-understood: run a small draft model to propose a block of tokens, verify them with the target model in a single forward pass, accept what matches, and correct where it doesn't. Done well, you get near-linear throughput gains on output-heavy workloads. Done poorly, you get a draft model that needs retraining every time the target model is updated, a separate model to load into memory, and acceptance rates that fall apart on distribution shifts.

[Orthrus](https://arxiv.org/abs/2605.12825), out of a team at arXiv this week, asks whether the draft head needs to be a separate model at all.

## Two Heads, One Cache

The core idea is that a frozen AR backbone already contains all the contextual representations you need to propose parallel tokens — they're sitting in the KV cache. Orthrus augments the backbone with a lightweight diffusion head (three new projection matrices: \(W_Q^\text{diff}\), \(W_K^\text{diff}\), \(W_V^\text{diff}\)) that attends to the same KV representations as the AR head, initialized from the AR counterparts. The diffusion head takes an anchor token plus \(K-1\) masked tokens and produces \(K\) candidates in a single forward pass. These candidates are then routed through the frozen AR head to get exact causal probabilities in a second pass.

The memory cost of this is constant: ~4.5 MiB regardless of sequence length. There's no second model's KV cache to maintain, no separate process, no model registry to keep in sync. Trainable parameters add up to about 16% of the backbone — and training requires less than 1 billion tokens on a single 8×H200 node.

## The Lossless Guarantee

What distinguishes Orthrus from diffusion-language-model approaches like MDLM or Fast-dLLM-v2 is the consensus mechanism. After the diffusion head proposes \(K\) tokens, each candidate is validated against the AR greedy prediction. The system commits accepted tokens and corrects the first mismatch with the exact AR output. For non-greedy (temperature > 0) generation, rejection sampling is applied to ensure strict alignment with the target distribution. The result: zero degradation on any benchmark. MATH-500 accuracy is identical to the baseline, GSM8K is identical, HumanEval is identical. You can verify this because the distribution is mathematically identical — the AR path is the arbiter, not an approximation.

This matters in practice. Existing speculative decoding systems with separate draft models get acceptance rates of perhaps 70–90% in the best case, and those rates drop when the model is fine-tuned or when the input distribution drifts. A deployment running EAGLE-3 against a base model has to retrain or re-calibrate the drafter when the backbone changes. Orthrus doesn't have this problem: the diffusion head was trained to agree with its host, and the consensus mechanism guarantees agreement at inference time regardless.

## Numbers

On Qwen3-8B, Orthrus achieves an average of 5.39 tokens per forward pass (versus the theoretical AR maximum of 1.0) and a mean 5.36× wall-clock speedup across coding, math, and reasoning tasks. The peak speedup of 7.8× occurs on tasks where the model tends toward repetitive or highly predictable outputs — exactly the case where long acceptance runs are most achievable. Compared to EAGLE-3 and DFlash, two strong speculative decoding baselines, Orthrus matches or exceeds acceptance lengths while eliminating the separate model overhead.

The authors test across Qwen3-1.7B, 4B, and 8B to validate that the approach scales within the model family. Results hold across sizes.

## What This Doesn't Solve

Orthrus requires two forward passes per generation step: one for the diffusion head to propose, one for the AR head to verify. The net speedup relies on these two passes being faster than \(K\) sequential AR passes — which is true when the batch is memory-bandwidth-bound (the common case on modern hardware), but the arithmetic changes on compute-bound workloads or very short \(K\). The paper doesn't characterize failure modes on compute-bound configurations.

Training the diffusion head also requires access to the backbone's internals at a fairly low level — this isn't a pure inference-time trick you can apply to a model you're querying via API. It's a modification requiring model weights.

None of this undermines the core result. The implicit cost of keeping a separate draft model calibrated, versioned, and in sync with the target model is real — every team operating a production serving stack with speculative decoding knows this. Orthrus's bet is that folding the draft into the backbone, at the cost of a modest training run, beats the ongoing operational overhead. Based on the numbers, it's a reasonable bet.

The code and Qwen3-8B checkpoint are [on GitHub](https://github.com/chiennv2000/orthrus).
