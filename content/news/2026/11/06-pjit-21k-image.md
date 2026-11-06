---
title: "Twenty-One Thousand Dollars, End to End"
date: 2026-11-06T06:00:00+01:00
draft: false
slug: pjit-21k-image
categories: [image-generation]
tags: [diffusion, open-source, training, efficiency]
params:
  author: AI Beat Desk
  summary: >-
    Linum released P-JiT, a 2B pixel-space text-to-image model trained from scratch for $21,038. The Apache 2.0 weights come with a full accounting of training choices, most notably the decision to drop the VAE entirely and work directly in pixel space with 32×32 patches.
---

[Linum published P-JiT (Pyramid-JiT)](https://www.linum.ai/field-notes/pjit-21k) on November 4, with a cost breakdown that actually holds up: $20,348 for pretraining and $690 for post-training, billed at $2.50 per H100-hour over 8,415 hours on 64 cards. The result is a 2-billion-parameter text-to-image model, Apache 2.0, trained mostly from scratch. The stated goal is not a production image tool but a research stepping stone toward video generation.

What makes it technically interesting is not the sticker price but the trade-offs that made that price achievable.

The most distinctive choice is working in pixel space without a VAE. Most text-to-image models compress images into a latent representation with a variational autoencoder, run diffusion there, and decode at inference. Linum skips both steps. Images are split into 32×32 patches directly, which gives four times fewer attention tokens than the 16×16 patch sizes common in earlier architectures—256 tokens per image at 512² resolution instead of 1,024. Fewer tokens means cheaper attention, cheaper training.

Text conditioning comes from a frozen Qwen3.5-4B, which sidesteps the cost of jointly training a text encoder from scratch. The backbone is 22 identical DiT trunk blocks, plus two refiner blocks each for text and image features. The design is deliberately unremarkable in structure; the novelty is in the training recipe.

Two training aids are worth noting. First, readout heads positioned at multiple points along the trunk predict low-resolution versions of the target image, giving useful gradient signal to early layers that would otherwise receive little feedback in a deep diffusion transformer. Second, a PixelREPA loss aligns the model's internal representations with those of DINOv3-L, borrowing perceptual structure from a large pretrained vision model without paying to generate it from scratch. Both choices address the same underlying problem: getting good gradients through a deep network when the training signal is diffuse.

The training curriculum started at 512² for 610 million samples, then shifted to 1024² for 50 million, with a noise-timestep schedule that widened after the first 80 million samples. Post-training applied a variant of Finite Difference Flow Optimization using a mix of an internal preference dataset and human judges.

The model's limitations are real. It cannot render text in images, struggles with full-body figures and multi-person compositions, and has no world knowledge in the sense of knowing what specific places or brands look like. Linum is transparent about these gaps, and they are largely predictable from the training choices—a 2B pixel-space model trained for \$21K is not going to match FLUX or Stable Diffusion 4.

What the release actually demonstrates is that the barrier to entry for image model training has dropped to something a small team can reason about concretely. 8,400 H100-hours at spot prices is not pocket money, but it is an auditable number, and the full architecture writeup makes it possible to understand exactly where that compute went. The weights and code under Apache 2.0 mean the design is forkable. The benchmarks are thin—only validation loss on a fixed 20,480-image eval set—so external comparisons are hard, but the architecture is documented well enough to replicate and extend.

Linum frames P-JiT explicitly as infrastructure for video work. Pixel-space diffusion generalizes to video more cleanly than latent-space approaches do—no encoder or decoder to retrain, and temporal consistency is easier to enforce when the model operates on actual pixels. Whether the bet pays off depends on how the video work goes, but as a documented proof-of-concept for cost-conscious image model training, the release is worth examining on its own terms.
