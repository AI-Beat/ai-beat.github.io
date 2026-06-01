---
title: "Image Generation at 1 Bit"
date: 2026-06-01T06:11:25+00:00
draft: false
slug: bonsai-image-1bit-diffusion
categories: [models]
tags: [models, inference, edge, quantization, image-generation]
params:
  author: AI Beat Desk
  summary: >-
    PrismML's Bonsai Image 4B applies 1-bit and ternary quantization to a
    FLUX.2 Klein diffusion transformer, compressing it 8.3× to 0.93 GB —
    small enough to generate images on an iPhone in under 10 seconds. It's
    the first demonstration that extreme quantization techniques developed for
    language models transfer cleanly to diffusion architectures.
---

The 1-bit quantization story for language models has been building for two years: [BitNet b1.58](https://arxiv.org/abs/2402.17764), then a series of follow-ons showing that ternary and binary weights, combined with per-group FP16 scale factors, can preserve most of the quality of full-precision transformers while slashing memory requirements. What [PrismML's Bonsai Image 4B](https://prismml.com/news/bonsai-image-4b), released May 26, demonstrates is that the same technique applies to diffusion transformers — and works well enough to be practically useful.

The base model is FLUX.2 Klein 4B, a compact text-to-image diffusion transformer with 4 billion parameters. At full FP16 precision the transformer weights alone occupy 7.75 GB, which rules out most mobile devices. PrismML's approach keeps the architecture unchanged but replaces the weights: the 1-bit variant uses binary {−1, +1} weights with a shared FP16 scale factor per group of 128 weights, yielding 1.125 effective bits per weight. The ternary variant adds a zero state, reaching 1.71 effective bits. About 5% of precision-sensitive layers — the projection matrices — stay in FP16.

The results are worth looking at directly. The 1-bit transformer weighs 0.93 GB, an 8.3× reduction. The ternary version is 1.21 GB, 6.4× smaller. On the [GenEval benchmark](https://arxiv.org/abs/2310.11513), which probes compositional image generation (counting, spatial relationships, color bindings), the full-precision model scores 0.819. The ternary model scores 0.723 — roughly 88% of the original. The 1-bit model scores 0.671. These aren't equal-quality outputs, but they're not junk either, and for on-device generation the tradeoff looks reasonable.

The deployment numbers are what make this interesting. The 1-bit model uses 1.5 GB of mean-active memory during 512×512 generation — down from 11.74 GB for the original — and generates that 512×512 image in 9.4 seconds on an iPhone 17 Pro Max, or about 6 seconds on a Mac M4 Pro. The full deployment payload including the compressed text encoder and FP16 VAE is 3.42 GB. PrismML says this is the first image model in its parameter class to run directly on an iPhone.

The technical reason this works is the same reason it works for language models: the transformer blocks in diffusion architectures don't need the same precision that training requires. Most of the signal is in the direction of the weights, not their exact magnitude, and the group-wise scale factors recover the magnitude information at low overhead. What's slightly more surprising is that the image decoder (the VAE) and text encoder were left in higher precision without being major bottlenecks — the transformer was the right place to apply quantization.

This matters beyond the benchmark numbers. FLUX.2 Klein is a capable model; the full-precision version produces decent images at reasonable prompt adherence. Getting that capability into a sub-1 GB package that runs on consumer hardware without any server infrastructure is a different kind of threshold than "runs in the cloud slightly cheaper." Weights and code are on Hugging Face under Apache 2.0, along with MLX and GemLite variants for Apple Silicon and mobile inference backends. There's also a Bonsai Studio iOS app for direct testing.

It's worth noting what this doesn't prove: that 1-bit diffusion at higher resolutions or with more demanding aesthetic quality targets is solved. GenEval at 512×512 is a fairly low bar for what people actually want from image generation. But it proves the transfer: the techniques work on this architecture, the compression ratio holds, and the quality degradation is bounded and predictable.
