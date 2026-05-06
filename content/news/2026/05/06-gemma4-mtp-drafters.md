---
title: "Gemma 4 Gets Speculative Decoding That Ships"
date: 2026-05-06T06:12:00Z
draft: false
slug: gemma4-mtp-drafters
categories: [models]
tags: [gemma, google, inference, speculative-decoding, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Google ships multi-token prediction draft models for the full Gemma 4 family
    under Apache 2.0, reporting up to 3x throughput gains. The architecture is
    tightly coupled — shared embeddings, last-layer activations — which keeps
    the drafter accurate but limits reuse. MoE variants complicate the picture.
---

Speculative decoding has been a known technique for improving LLM inference throughput since at least 2023. The idea is simple: use a cheap draft model to predict several tokens ahead, then let the full target model verify them in parallel. If the draft tokens are accepted, you get multiple tokens for roughly the price of one target model pass. The catch is that a bad draft model wastes effort on proposals the target model rejects, and building a good draft model that generalizes is harder than it sounds.

Google [released multi-token prediction drafters for the full Gemma 4 family](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/) yesterday, under the same Apache 2.0 license as Gemma 4 itself. The headline number is up to 3x throughput without degradation in output quality. That's consistent with what good speculative decoding implementations achieve on favorable workloads.

The architecture is more interesting than the number. These drafters aren't independent models trained separately. They share the target model's embedding table entirely, and they build on the target model's last-layer activations — meaning the drafter takes the target model's own internal state as part of its input. The sequence is: target model processes context, drafter takes those final-layer activations, concatenates them with token embeddings, down-projects to its own working dimension, and predicts several tokens ahead. The target model then verifies the drafts in a single parallel pass.

The tight coupling matters for a few reasons. Because the drafter sees the target model's own activations, it has rich signal about what the target model is about to do next — it's essentially getting the target model's view of the context for free. That tends to produce better acceptance rates than a fully separate draft model that has to infer context from the token sequence alone. The cost is that this drafter only works with this specific target model; you can't reuse it with a different Gemma 4 variant or another model family.

For the E2B and E4B edge variants, Google added an additional optimization: a clustering technique in the embedder that groups similar tokens, restricting the drafter's final projection to only the tokens in the selected cluster rather than scoring the full vocabulary. This reduces compute on already-constrained edge hardware.

The MoE story is more complicated. Gemma 4 26B is a mixture-of-experts model (26B total parameters, 4B active per token). The drafter for the 26B A4B exists, but the [documentation](https://ai.google.dev/gemma/docs/mtp/overview) hedges: because different tokens activate different experts, the MoE model's last-layer activations are less predictable in structure than a dense model's, and at batch size 1 on Apple Silicon specifically, the 26B A4B drafter may yield no speedup at all. Speculative decoding and MoE interact poorly in general — MoE models are fast because they compute less per token, which reduces the gap that drafting can exploit, and the routing variance makes draft quality harder to maintain across diverse inputs.

The drafters are available on Hugging Face and Kaggle and work with `transformers`, MLX, vLLM, SGLang, and Ollama today. The practical impact is immediate for anyone running Gemma 4 dense models locally or in self-hosted infrastructure: 2x–3x throughput means either faster responses at the same compute budget, or equivalent response speed at lower hardware requirements. For edge deployments on Apple Silicon, Google reports around 2.2x average speedup when handling concurrent requests.

What's notable here is less the technique itself — speculative decoding has appeared in open-source inference engines for a while — and more that Google is packaging it as a first-class, model-specific artifact tied to a specific release. The drafter ships alongside the model weights, targets specific hardware profiles, and integrates directly with the major inference frameworks. That's a distribution story as much as a research one. Optimization techniques that ship alongside the model weights at launch tend to actually get used.
