---
title: "What $998 Buys You"
date: 2026-09-10T06:11:47+00:00
draft: false
slug: little-lm-benchmark-context
categories: [training]
tags: [training, benchmarks, efficiency, small-models, evaluation]
params:
  author: AI Beat Desk
  summary: >-
    Hugo Vergnes trained a 3.8B-parameter LLM for $998 in compute costs, scoring
    0.384 on the CORE benchmark. The interesting result isn't the price — it's
    that doubling context length from 1024 to 2048 tokens drove 83% of the score
    improvement on one key task, which says something about what benchmark numbers
    are actually measuring.
---

[Hugo Vergnes published a write-up today](https://hugovergnes.github.io/little-lm-3-8b/) of training a 3.8B-parameter decoder-only LLM from scratch for $998 in compute costs, 43 hours on rented B200 GPUs, and 65 billion training tokens. The model scores 0.384 on the [CORE benchmark](https://hugovergnes.github.io/little-lm-3-8b/), meaningfully above GPT-2's 0.2565. It's a clean, well-documented solo experiment in what current techniques get you at modest scale.

The $998 number is the hook, but it's not the finding. The finding is buried in the benchmark analysis.

On SQuAD — a reading comprehension task — the model scored 0.0000 at a 1024-token context window. Expanding to 2048 tokens pushed that to 0.3114. Vergnes notes this context-length sensitivity accounts for 83% of the score improvement between context lengths. One configuration choice, no changes to model weights, turns a zero into a respectable score.

This is more revealing than it sounds. SQuAD asks questions about passages of text. If your context window is too small to hold both the question and the relevant passage, you cannot answer correctly — not because you lack the capability, but because the prompt is physically truncated. The model isn't failing; the evaluation setup is failing the model. Scores reported for a model on SQuAD are silently conditional on context window size, and the relationship isn't smooth: you can be at 0.0000 and then jump to 31% with a single configuration change.

Benchmark numbers routinely carry implicit assumptions about tokenization schemes, prompt formats, context window settings, and decoding strategies. The SQuAD/context-length case is a clean example because the failure mode is obvious: the passage doesn't fit. In other cases the dependency is subtler — a different few-shot prompt format, a different sampling temperature — and the result difference may be several percentage points in either direction without any explicit disclosure. When you see comparative benchmark numbers from different groups, you're often comparing configurations that differ in ways the comparison table doesn't capture.

The modern training stack Vergnes used is worth noting separately. The optimizer is [Muon](https://github.com/KellerJordan/modded-nanogpt) for matrix parameters and AdamW for everything else — a combination that's become a practical alternative to Adam-only training at this scale. Training uses FP8 with dynamic tensorwise scaling for memory efficiency. The learning rate schedule is trapezoidal rather than cosine decay. The dataset is [ClimbMix](https://arxiv.org/abs/2505.01824) rather than FineWeb-Edu. All of these are techniques that were either research curiosities or proprietary a few years ago; they're now documented well enough that a solo practitioner picks them up from blog posts and implements them over a weekend.

The model architecture itself is a Llama-style decoder with grouped query attention (24 query heads, 8 KV heads), RoPE positional embeddings, RMSNorm, and non-gated ReLU² MLPs. Nothing exotic — the purpose was to test training efficiency and dataset quality rather than to push architectural novelty.

What the experiment demonstrates is roughly: a single person with intermediate knowledge of the field, renting GPU time for the cost of a decent laptop, can now train a model that substantially outperforms the standard baseline (GPT-2) using techniques published in the past year. The gap between "what major labs do" and "what an individual can replicate" has narrowed mostly on the technique side. Data curation and raw compute budget remain where the gap is biggest — Vergnes trained on 65B tokens, where frontier models typically see tens of trillions.

The project is called [little-lm](https://github.com/hugovergnes/little-lm) and is config-driven: each training run is specified in a YAML file with no code changes required. That's a practical convenience for iterating on hyperparameters, and it makes the experiment reproducible by construction — anyone can clone the repo, point at their own compute budget, and run the same training process.

The benchmark fragility point generalizes beyond this one experiment. The context-length dependency on SQuAD is a clean data point for an argument worth keeping in mind whenever a model comparison table lands in your feed: before interpreting the numbers, it's worth asking what context window, what prompt format, and what decoding configuration produced them.
