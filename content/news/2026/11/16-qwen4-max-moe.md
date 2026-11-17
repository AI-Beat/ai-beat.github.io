---
title: "Qwen4-Max and the N-gram Embedding Gamble"
date: 2026-11-16T06:13:00+00:00
draft: false
slug: qwen4-max-moe
categories: [models]
tags: [qwen, alibaba, moe, multimodal, inference]
params:
  author: AI Beat Desk
  summary: >-
    Alibaba's Qwen4-Max lands today with a 2.4T-parameter MoE backbone, 71B
    active per token, and a 328B N-gram Embedding component Alibaba treats as a
    first-class architectural addition. The model claims 50% fewer tokens per
    task than its predecessor and ships with native vision and a 1M-token
    context window—closed weights, though smaller open Qwen4 variants are
    promised.
---

Alibaba's [Qwen4-Max](https://qwen.ai/blog?id=qwen4-max) arrived today, and the headline numbers are the kind that prompt a double-take: 2.4 trillion backbone parameters with only 71 billion activated per token, plus a separate 328B N-gram Embedding component Alibaba counts alongside the main model rather than folding into the parameter totals.

That last piece is the architecturally unusual bit. N-gram embeddings are not new—they go back to subword pre-tokenization tricks and some hybrid retrieval ideas—but attaching a 328-billion-parameter embedding table to a sparse MoE reasoning model and treating it as a first-class upgrade in a flagship announcement is a different kind of commitment. Alibaba says the new Qwen4 architecture "systematically upgrades" the sparse MoE across attention, residuals, and embeddings, and the N-gram component appears to be a large chunk of what that means in practice.

The practical results, as Alibaba measures them: approximately 5–13% lower bits-per-byte on the base model versus Qwen3.8-Max-Base while activating 25% fewer parameters; 48–49% fewer tokens per task on the Terminal-Bench 4.0 coding and tool-use benchmark; and a jump from 47.74 to 52.69 on Work360, Alibaba's own multi-domain work-tasks suite. That's roughly a 10% improvement on a benchmark the company designed, so independent confirmation will matter. Still, the token-efficiency claim is specific and testable, and "fewer output tokens for the same quality" is the kind of improvement that shows up directly in costs.

On context and modality: the model accepts text, images, and video natively, with a 1M-token context window and reasoning effort that can be set to low, high, or xhigh. The API is compatible with both OpenAI Chat Completions and Anthropic Messages formats, which is table stakes for any model aiming at the agent-framework market. Pricing is $1.50 per million input tokens and $4.50 per million output, with cached input at $0.075.

The obvious gap: no open weights. Alibaba says smaller Qwen4 series models will be released to developers, but nothing has shipped yet. The Qwen line has historically offered strong open-weight variants that ended up powering a large fraction of the community's fine-tuning experiments, so the eventual release of a Qwen4-72B-style model will matter more than the flagship announcement for most practitioners. For now, Qwen4-Max is an API product competing directly with the closed tiers of Gemini 4, GPT-5.x, and Claude Fable.

One thing worth watching: if the N-gram Embedding claim holds up, it suggests Alibaba found a way to pack in vocabulary-level pattern retrieval without the latency and infrastructure overhead of a separate retrieval system—essentially caching n-gram statistics into a large embedding lookup that runs at inference time. Whether that generalizes to arbitrary domains or is tuned tightly to Alibaba's internal benchmarks is not clear from the announcement. The architectural choice is interesting enough that it deserves scrutiny once the technical report arrives.
