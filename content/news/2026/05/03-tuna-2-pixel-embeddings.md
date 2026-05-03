---
title: "Drop the Encoder: Meta's Tuna-2 Goes Straight to Pixels"
date: 2026-05-03T06:21:22+00:00
draft: false
slug: tuna-2-pixel-embeddings
categories: [research]
tags: [multimodal, vision, meta, architecture, training]
params:
  author: AI Beat Desk
  summary: >-
    Meta AI's Tuna-2 paper shows that a 7B unified multimodal model trained
    end-to-end on raw pixel patches — with no pretrained vision encoder — matches
    or beats its CLIP-based sibling at scale, particularly on fine-grained
    perception tasks. The result challenges a design assumption that has been
    stable in multimodal modeling for years.
---

The standard blueprint for a multimodal model has been stable for a while: take a pretrained vision encoder (usually CLIP or a derivative), a language model, glue them with an adapter, and fine-tune. The vision encoder handles visual representation; the LLM handles language. The two halves were trained separately before they ever met.

[Tuna-2](https://arxiv.org/abs/2604.24763), a new paper from Meta AI, asks whether the encoder half is load-bearing. The answer, at the 7B scale with 550M training images, appears to be no — or at least, not as obviously as the field has assumed.

The model uses Qwen2.5-7B-Instruct as its language backbone and replaces the vision encoder with simple patch embedding layers applied directly to raw pixel patches. No CLIP, no SigLIP, no VAE. The image is tiled, each patch is linearly projected into the token stream, and the full transformer handles the rest. The architecture is simpler than most multimodal models by a significant margin, which matters for end-to-end gradient flow: every parameter from the pixel patch to the final output can be optimized together, without the representation bottleneck imposed by a frozen encoder trained on a different objective.

The result on understanding benchmarks is that Tuna-2 outperforms its encoder-based sibling (Tuna-R, which uses a pretrained vision encoder) on GQA (65.0 vs. 63.5) and OCRBench (79.7 vs. 78.3), with the gap growing on tasks requiring fine-grained visual perception. On generation, Tuna-R has a slight edge early in training, but the gap essentially closes after supervised fine-tuning — GenEval scores land within rounding error of each other (0.87 for Tuna-2).

One pattern worth noting from the ablations: encoder-based models converge faster in early pretraining. This makes sense — a pretrained CLIP encoder already contains useful visual structure, so you're not starting from scratch. The encoder-free model closes the gap as training compute increases. The implication is that the apparent strength of CLIP-based approaches is partly a compute argument, not an architectural one: at the compute scales most papers use, pretrained encoders are a head start. At larger scale, that head start matters less.

There are caveats. The paper trains on 550M proprietary image-text pairs, which means the result isn't straightforwardly replicable on public data. The model scale is 7B, which is solid but not frontier. And "state-of-the-art on multimodal benchmarks" is a phrase that appears in most multimodal papers; the specific numbers here are competitive rather than dominant.

What the paper does establish is that the field's default assumption — that pretrained vision encoders are a required component of a capable multimodal system — has less empirical support at moderate scale than the prevalence of the approach implies. That's worth knowing. Training a single model end-to-end from pixels simplifies the development pipeline, avoids alignment problems between separately-trained components, and lets the model develop visual representations shaped by the task rather than by CLIP's contrastive objective. If Tuna-2's results hold as model size grows, the two-part encoder+LLM design may start looking more like a historical artifact than a principled choice.

The [code is on GitHub](https://github.com/facebookresearch/tuna-2) under Apache 2.0. Full production weights are not released due to organizational policy constraints, but the team says they plan to release a foundation checkpoint — with some layers removed — that can be fine-tuned on external data to restore full functionality.
