---
title: "Dropping the Encoder"
date: 2026-05-14T06:15:00+00:00
draft: false
slug: sensenova-u1-dropping-the-encoder
categories: [models]
tags: [multimodal, architecture, open-source, generation, understanding, vision]
params:
  author: AI Beat Desk
  summary: >-
    SenseTime's SenseNova-U1 open-sources a unified multimodal model that removes
    both the visual encoder and VAE — the two architectural crutches that every
    major multimodal system has relied on since the CLIP era. The NEO-unify
    architecture processes pixels natively through a shared transformer backbone,
    with a direct pixel-space MLP head for generation. Benchmarks on image
    generation and interleaved content put it at or above current open-source
    leaders, with the spatial reasoning numbers being the most credible
    differentiator.
---

The standard recipe for building a multimodal model has looked roughly the same for four years. You take a pretrained vision encoder — CLIP, SigLIP, or something similarly trained on image-text pairs — and wire it to a language model. For image generation you add a variational autoencoder or diffusion decoder on the other end. The two halves share little except the interface between them: the vision encoder produces embeddings, the LLM interprets them, and the generative half translates latent codes back to pixels. Every major lab doing serious multimodal work has followed some version of this blueprint.

[SenseNova-U1](https://arxiv.org/abs/2605.12500), released by SenseTime last month with the technical report out this week, makes a different bet. The NEO-unify architecture throws away both the visual encoder and the VAE. There is no pretrained CLIP-style component; there is no latent space managed by a separate encoder-decoder. Images enter as raw pixels, mapped to token sequences by a two-layer convolutional patch encoder with strides of 16 and 2, so each visual token represents a 32×32 image patch. On the output side, a small MLP head predicts pixel patches directly, without a VAE decoder in the loop.

What remains is a single backbone — a Mixture of Transformers — processing both modalities. Text tokens attend causally (standard left-to-right language modeling); image tokens attend bidirectionally. The two-stream design keeps understanding and generation parameters partially decoupled (separate projections, normalizations, and feedforward blocks per stream) while sharing the underlying architecture.

## Why This Is a Reasonable Bet

The pretrained visual encoder provides representation learning that took enormous compute to acquire. Dropping it means you have to learn visual representations from scratch as part of the same training run that teaches the model to reason and generate. That's expensive, and it's why most efforts to go "encoder-free" have struggled.

The counterargument, which SenseNova-U1 is making implicitly, is that pretrained encoders impose a ceiling. CLIP was trained to align images and captions as a retrieval task; its internal representations optimize for that objective, not for fine-grained spatial reasoning, not for generation, not for agentic decision-making. When you bolt a CLIP encoder onto a generalist model, you're importing the biases and the blind spots that CLIP learned alongside its strengths. The representation space is fixed, and it was shaped by a different task.

A native architecture, in contrast, has a single coherent learning signal. The same gradient that trains the model to understand an image also trains it to generate one. Understanding and generation reinforce each other rather than coming from separately trained components connected by an adapter layer.

Whether that's worth the training cost is an empirical question, and the results are mixed but suggestive.

## The Benchmark Numbers

On generation quality, [GenEval](https://github.com/djghosh13/geneval) is the clearest number: SenseNova-U1 (both variants) scores 0.91, compared to 0.87 for Qwen-Image and 0.82 for BAGEL. That's a real margin on a benchmark that tests compositional generation ability — getting object counts right, attribute binding, and spatial relationships in generated images.

For interleaved text-image generation — producing alternating prose and images in a coherent sequence — SenseNova-U1-A3B scores 9.16 on OpenING with chain-of-thought enabled. That puts it above GPT-4o paired with DALL-E 3 (8.20) and Wan-Weaver (8.67). This is arguably the most interesting result because interleaved generation is where the representation unification story makes the most sense: a model that jointly understands and generates doesn't have to translate between two separate internal vocabularies.

The spatial reasoning numbers are also interesting. On [VSI-Bench](https://arxiv.org/abs/2409.17490), which tests visual-spatial intelligence (estimating distances, sizes, and object relationships from images), the 8B variant scores 62.66 against 55.67 for Qwen3.5 9B. Spatial reasoning is exactly the kind of task where the CLIP encoder's training objective offers the least help — CLIP learned to match images to captions, not to estimate distances — so it's plausible that a native architecture does better here.

Understanding benchmarks tell a more ambiguous story. On MMMU the model scores 74.78 against Qwen3.5 9B's 78.40, and on OCRBench there's a visible gap (82.10 vs 89.20). The knowledge-intensive and text-heavy tasks remain harder without the pretrained representation head start.

## The Technical Detail Worth Noting

The inference pipeline uses distribution matching distillation to reduce the generation process from 100 steps to 8 steps without meaningfully degrading quality. At 2048×2048 on an L40S GPU that comes out to about 0.44 seconds per step. This matters for practical deployment: 8 diffusion steps is the difference between generation that blocks a UI and generation that doesn't.

The A3B variant is also worth noting as an engineering choice. It has 30B total parameters across 128 experts for the understanding stream, but only about 3B active at inference time. That keeps inference costs modest while giving the model access to wider representations during training.

## What This Doesn't Settle

Open-sourcing a model with competitive benchmark numbers doesn't prove that the architectural approach is correct. There are other explanations: SenseTime may have simply trained longer, on better data, with a better post-training recipe. The benchmark suite, while reasonable, doesn't test everything relevant to real deployments. And the gap on understanding tasks leaves open whether the representation learning from scratch actually converges to something as good as leveraging decades of pretrained visual supervision.

The interesting follow-up question is whether the unified gradient signal demonstrably helps — whether you can find tasks where understanding transfers to generation or vice versa in ways that a mixed-component system can't match. That would require controlled ablation experiments, not a single model release. The [NEO-unify blog post](https://huggingface.co/blog/sensenova/neo-unify) and [technical report](https://arxiv.org/abs/2605.12500) make the architectural argument clearly but leave the ablation evidence limited.

For now, SenseNova-U1 is the most concrete public implementation of the encoder-free thesis, and the generation quality numbers are strong enough to take seriously. Whether the approach generalizes as a training paradigm is a question that will need more evidence.
