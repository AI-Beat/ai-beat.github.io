---
title: "Tencent's Hy4 Is a Very Aggressive MoE"
date: 2026-08-30T06:12:56+00:00
draft: false
slug: tencent-hy4-preview-moe
categories: [models]
tags: [open-source, moe, inference, architecture]
params:
  author: AI Beat Desk
  summary: >-
    Tencent released Hy4 preview under Apache 2.0 on August 28: 770B total parameters,
    49B active, 256 routed experts per layer with top-8 routing, 1M context, and
    a built-in speculative-decoding layer. The architecture makes some interesting
    bets — and the ratio of total to active compute is the most aggressive we've
    seen at this scale.
---

Tencent's [Hy4 preview](https://www.tencent.com/tencent-releases-and-open-sources-tencent-hy4-preview/), released August 28 under Apache 2.0, is now on [Hugging Face](https://huggingface.co/tencent/Hy4-preview) and available through vLLM and SGLang. The headline numbers are 770B total parameters and 49B active — a 15.7:1 ratio that is notably more aggressive than anything else at this scale in the open-source ecosystem.

## What 256 experts per layer actually means

Most MoE models run 8–64 experts per layer and activate 1–4 of them. Hy4 runs 256 routed experts per layer plus 1 shared expert, and activates the top-8 routed experts. The first of 78 layers uses a standard dense FFN; the remaining 77 layers are all MoE.

This is a different design philosophy. With 256 choices and 8 selected, the router has much finer granularity — it can specialize experts far more narrowly, but it also pushes load-balancing complexity and expert-routing overhead to the extreme. Whether this actually translates to better generalization per activated FLOP than a coarser MoE is an empirical question the architecture is making a bet on.

The hidden size is 6144 with 64 attention heads, which is on the smaller side for the total parameter count.

## Sparse attention and inter-layer plumbing

The attention module uses [Gated DeepSeek Sparse Attention (Gated DSA)](https://github.com/Tencent-Hunyuan/Hy4-preview) with IndexCache for cross-layer sparse index reuse. The idea, borrowed from DeepSeek's attention work, is to reuse the set of "which positions are important" across layers rather than recomputing it fresh each time — saving both compute and memory bandwidth in long-context settings.

The residual stream uses iHC (identity Hyper-Connections), a technique for expanding inter-layer information flow by letting each layer read from multiple prior layer outputs rather than just the immediately preceding one. This is a relatively niche architectural ingredient and one of the less-common choices in the model.

There's also a native MTP (Multi-Token Prediction) layer with 10B total and 0.7B active parameters, built in for speculative decoding. Speculative decoding support being baked into the architecture rather than retrofitted is a practical deployment choice — it means the latency gains from speculative decoding come without the usual tuning overhead of finding a compatible draft model.

## How Tencent measured it

The evaluation is interesting primarily for what it isn't: Hy4's performance claims rest mainly on an internal blind evaluation by 163 Tencent employees across 203 engineering tasks drawn from real work in software, gaming, finance, and research. The model scored 2.99/4.00 versus 2.92/4.00 for GLM-5.3 and 2.94/4.00 for Kimi K3. That's a very thin margin, and the setup is not independently reproducible.

Public benchmark coverage is thinner than you'd expect from a model at this scale. That's a deliberate choice — the team says training data was centered on the work Tencent's own experts actually do, prioritizing productivity on realistic tasks over optimizing standard leaderboards. Whether the internal scores generalize is the open question.

The known weaknesses Tencent lists honestly: the model tends toward verbosity in reasoning and over-verification. They preferred early release for feedback over waiting for a cleaner result, which is a reasonable position.

## Deployment

Weights are out on [Hugging Face](https://huggingface.co/tencent/Hy4-preview) in both BF16 and [FP8-quantized](https://huggingface.co/tencent/Hy4-preview-FP8) variants. The FP8 version drops the serving memory requirement substantially while preserving most of the model's performance — at 49B active parameters even the BF16 version demands serious hardware, but FP8 makes it feasible on a pair of high-memory GPUs rather than requiring a full GPU cluster.

vLLM and SGLang both support it natively as of their latest builds, so the standard serving infrastructure works without patching.

Tencent notes the model was trained on domestically produced Chinese chips, which is relevant context for anyone tracking the supply-chain story around frontier training hardware.

At Apache 2.0 with FP8 weights available, this is one of the more accessible open-weight releases at the 49B-active-parameter tier. Worth running evals against your actual workload if productivity tasks on long documents are in scope.
