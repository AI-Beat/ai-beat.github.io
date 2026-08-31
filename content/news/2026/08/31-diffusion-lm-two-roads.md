---
title: "Two Roads Through Diffusion Space"
date: 2026-08-31T06:13:04+00:00
draft: false
slug: diffusion-lm-two-roads
categories: [research]
tags: [diffusion-models, language-models, inference, efficiency, training]
params:
  author: AI Beat Desk
  summary: >-
    Two tutorial posts on Hacker News today trace the two distinct technical lineages
    of diffusion language models — discrete masked and continuous embedding-space — both of
    which have now reached production deployments with 2–10× throughput gains over
    autoregressive models. The quality gap with AR is measurably narrowing.
---

Two posts about diffusion language models surfaced on Hacker News today, and they happen to cover opposite ends of the design space — one practical tutorial on building discrete masked DLMs, one historical and theoretical overview of continuous DLMs. Together they make a good map of where things stand.

The [Kuleshov Group's tutorial](https://kuleshov-group.github.io/blog/blog/2026/how-to-build-a-diffusion-language-model/), adapted from workshop talks at ICLR 2026 and MLSS 2026, covers the masked diffusion approach. The core idea is to treat token masking as the noise process: start with a fully masked sequence, train a bidirectional transformer to recover the original tokens. What makes this work is that the training objective collapses to something already well understood — a cross-entropy loss between the unmasking predictions and the true tokens, averaged across all masking rates. That's just the BERT loss. The surprise is that any masked language model you've already trained is, in a precise sense, already a diffusion model.

At inference you start from a fully masked sequence and iteratively unmask. The twist that makes it practical is *remasking*: after each prediction step, you re-mask some tokens and predict again. This creates a kind of error-correction loop that the left-to-right autoregressive process doesn't have. Tokens predicted too early can be revised. The throughput gains come from the fact that many tokens can be predicted in parallel in each step, rather than strictly one at a time. Production deployments show this clearly: Mercury 2 from Inception Labs reports over 1,000 tokens per second per user on standard GPUs, and Nemotron Diffusion at 35B parameters achieves 2–8× the throughput of comparable autoregressive models.

[Sander Dieleman's post](https://sander.ai/2026/08/24/continuous-dlms.html) takes the other road. Continuous DLMs predate masked diffusion — the idea is to embed discrete tokens in a continuous vector space and apply ordinary Gaussian noise there, inheriting the full machinery developed for image and audio generation. This approach was promising early but ran into trouble: while training was feasible, getting the models to match autoregressive quality proved harder than expected, and distillation — compressing a multi-step sampler into a few steps — was difficult enough that the throughput advantage evaporated in practice.

What changed is what Dieleman wrote about in May: [flow maps](https://sander.ai/2026/05/06/flow-maps.html). A flow map directly predicts any point along the diffusion trajectory from any other point, making the path between noise and data a navigable object rather than an ODE you have to integrate numerically. This makes distillation tractable in a principled way — single-step generation becomes achievable without the quality collapse that plagued earlier compression approaches. As Dieleman puts it, flow map methods "enable even single-step models to capture all correlations, at least in theory."

The two lineages have different characters. Discrete masked diffusion is simpler to understand and train — the BERT connection means there's no conceptual gap between pre-training and diffusion, and the categorical output distribution is clean. Continuous diffusion has a tighter connection to the broader diffusion model literature and may generalize more naturally to settings that mix modalities, where not all inputs are discrete tokens. In practice both are producing real throughput numbers that autoregressive models can't match at equivalent quality, and the quality gap itself is closing. DiffusionGemma [hit AIME 2026 at 69.1%](https://ai-beat.github.io/news/2026/06/diffusiongemma-text-diffusion/) against the autoregressive Gemma 4's 88.3% — not parity, but a narrower gap than most people expected a year ago.

What's interesting about today's two posts isn't any single new result. It's that both were written as *pedagogical* material — tutorials adapted for workshops, historical surveys for practitioners. That's the signal. You write tutorials when something has stabilized enough to teach. Diffusion language models have been in the "interesting but experimental" category for long enough that it's easy to miss that Mercury 2 is shipping to real users right now. The research community, apparently, thinks the moment to teach this to a general ML audience has arrived.

Whether the throughput advantage survives as autoregressive serving infrastructure continues improving (speculative decoding, flash attention variants, MLA) is a fair question. But the fact that multiple production systems are running at >1000 tok/sec on standard hardware, with quality within striking distance of the AR frontier, is a different status than this research area held twelve months ago.
