---
title: "Needle: What a 26M-Parameter Model Says About Tool Calling"
date: 2026-05-13T06:17:03+00:00
draft: false
slug: needle-26m-tool-calling
categories: [models]
tags: [on-device, function-calling, architecture, open-source, distillation, inference]
params:
  author: AI Beat Desk
  summary: >-
    Cactus Compute released Needle, a 26M-parameter MIT-licensed model for
    on-device function calling that strips out all feed-forward networks from the
    transformer. The architectural choice is a thesis: tool calling is
    retrieval-and-routing, not reasoning, and attention is the right primitive
    for it. The numbers are striking — 6000 tok/s prefill on consumer hardware —
    even if the playground has rough edges.
---

The standard assumption when people talk about shrinking AI agents for edge deployment is that you take a capable model and compress it: quantize, distill, prune. The result is always a scaled-down version of the general-purpose thing, with general-purpose weaknesses just smaller.

[Needle](https://github.com/cactus-compute/needle), released by Cactus Compute on May 12, takes a different position. It doesn't try to be a small general model. It's a 26M-parameter model designed specifically for function calling — and it removes the feed-forward networks from the transformer entirely.

## The Architecture Argument

Standard transformers alternate between attention layers and feed-forward (FFN) layers. The FFN accounts for roughly two-thirds of parameters in most transformer designs; it's where much of the per-token factual recall and feature transformation happens. Needle replaces this with additional attention layers and gated residuals: instead of `x = x + Attn(Norm(x))`, it uses `x = x + sigmoid(gate) * Attn(Norm(x))`, with learnable per-sublayer gating initialized at 0.5. The normalization scheme, ZCRMSNorm, initializes gamma to zero so each block starts as an identity transformation.

The theoretical case for this is laid out in [their architecture notes](https://github.com/cactus-compute/needle/blob/main/docs/simple_attention_networks.md): tool calling is fundamentally about routing — matching a query to a tool schema and extracting parameter values. That's a structural task, and attention (especially cross-attention) is purpose-built for it. FFNs, which store per-token feature representations accumulated from pretraining, are doing work that simply isn't needed if you're not doing open-ended language generation.

The 12-encoder / 8-decoder layout with 512 dimensions and 8 attention heads (4 KV heads) targets this regime specifically: sub-50M parameters where attention layers yield more signal per parameter than FFNs do on structured tasks.

## Training and Performance

Pretraining: 200B tokens across 16 TPU v6e chips, 27 hours. Post-training: 2B synthetic function-calling tokens distilled from Gemini-generated data, 45 minutes. The weights and dataset are on Hugging Face under MIT license.

On production hardware via Cactus's own inference engine, the model runs at 6000 tokens/second prefill and 1200 decode. At those speeds, the overhead of doing structured argument extraction becomes negligible for real-time agent loops. A wearable or phone that issues tool calls ten times per second isn't bottlenecked by inference; it's bottlenecked by the tools themselves.

In benchmarks, Needle reportedly outperforms FunctionGemma-270m and Qwen-0.6B on single-shot function-calling tasks, which are both models an order of magnitude larger.

## What Doesn't Work Yet

Early testers in the [HN thread](https://news.ycombinator.com/item?id=48111896) found rough edges. One person tried "contact my boss" and got a timer instead of an email lookup. The model appears to be sensitive to exact phrasing, which is predictable given that it was post-trained on synthetic data with a particular vocabulary of function names and argument patterns. Whether it generalizes gracefully to arbitrary real-world tool schemas is a real open question.

There's also the distillation question. Needle was trained on synthetic data generated using Gemini — and Google's terms of service prohibit using Gemini outputs to train competing models. Cactus's response is that they didn't access model weights and the output is sufficiently transformed, but this is a genuine gray area that has affected other open-source efforts in this space.

## What It Points To

The interesting thing about Needle isn't just the performance numbers. It's the architectural commitment. Removing FFNs entirely is a strong claim: that for this task class, the standard transformer design is carrying dead weight.

If that bet is right, it has implications for how people think about deploying function-calling agents on devices where a full Qwen-7B or Llama-3-8B isn't practical. You don't need a general reasoner that also knows how to call functions — you need a purpose-built routing layer that sits in front of whatever reasoning layer handles the hard parts. The cloud handles thinking; the device handles dispatch.

Whether Needle actually works well enough in practice to fill that role is something that will take real integration testing to determine. But the architecture question is worth taking seriously. General-purpose models made transformers ubiquitous. It's not obvious they're the right tool for every component in an agent system.
