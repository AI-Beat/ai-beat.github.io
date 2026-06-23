---
title: "The Inpainting Model That Skipped the Attention"
date: 2026-06-23T06:05:00+00:00
draft: false
slug: moebius-inpainting-0.2b
categories: [models]
tags: [diffusion, inpainting, efficiency, knowledge-distillation, vision]
params:
  author: AI Beat Desk
  summary: >-
    HUST's Moebius (0.22B) matches FLUX.1-Fill-Dev (11.9B) on six image inpainting
    benchmarks at 15× the inference speed. Two mechanisms make it work: Local-λ Mix
    Interaction blocks that replace quadratic spatial attention with fixed-size linear
    matrices, and adaptive multi-granularity latent-space distillation. For inpainting
    specifically, attention overhead appears to be the actual bottleneck — not parameter
    count. Weights are out.
---

The standard narrative around diffusion inpainting has been that you need large models to get convincing results. [FLUX.1-Fill-Dev](https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev), at 11.9 billion parameters, is the benchmark most recent work compares against. [Moebius](https://arxiv.org/abs/2606.19195), from the vision lab at Huazhong University of Science and Technology, makes a strong case that the relationship between parameter count and inpainting quality is less fundamental than it looks.

At 0.22 billion parameters — less than 2% of FLUX.1-Fill-Dev's count — Moebius matches or surpasses FLUX on six benchmarks covering natural and portrait scenes, and runs at 26 milliseconds per step: more than 15 times faster than 10B-scale models. The paper is accepted to ECCV'26; code and weights landed on [GitHub](https://github.com/hustvl/Moebius) on June 18.

The question worth asking is what FLUX is spending 11.9B parameters on that Moebius doesn't need.

## How it works

Inpainting is a constrained generation problem. You have a context image and a mask; the model fills the mask in a way that's coherent with what surrounds it. Full spatial self-attention in a standard diffusion model scales quadratically with sequence length, but inpainting has structure that makes most of that computation redundant. The model doesn't need to attend to every pair of spatial positions — it mainly needs to capture context near the mask boundary and the global semantic content of the image.

Moebius's first contribution, the **Local-λ Mix Interaction (LλMI) block**, exploits this. Rather than computing full spatial attention, it condenses spatial context and global semantic priors into fixed-size linear matrices — a constant-size representation regardless of image resolution. This restructures the diffusion backbone to bypass the O(n²) attention computation while keeping the information flow that actually matters for coherent inpainting.

The second piece is what gives a 0.22B model the capacity to match an 11.9B teacher: **adaptive multi-granularity distillation**. The student learns from FLUX in latent space rather than pixel space, which avoids the cost of repeated decoding during training. Crucially, the supervision operates at multiple levels simultaneously — from low-level texture features to high-level diffusion trajectory alignment — and the balance between those levels adjusts dynamically rather than staying fixed. The result is that Moebius absorbs representational knowledge from the teacher without needing the teacher's capacity at inference time.

## The result in context

Six evaluation sets across two domains (natural scenes and portraits) is enough to suggest the result isn't an artifact of a narrow test set. ECCV peer review presumably stress-tested the comparison against FLUX.1-Fill-Dev, and the numbers held.

Some skepticism is warranted about generalization. Inpainting benchmarks measure coherence at the mask boundary and semantic consistency in controlled settings; failure modes at high mask ratios, unusual textures, and complex occlusions sometimes surface only in deployment. The paper's two-domain coverage doesn't fully answer those questions.

The more interesting question is whether the LλMI approach generalizes beyond inpainting. If the argument is that full spatial attention is overcapacity for constrained generation tasks — where the model already knows the structure it needs to respect — the same logic could apply to outpainting, conditional generation with geometric constraints, or reference-based stylization. Those would be worth watching for in follow-up work.

For now, the practical implication is clear enough: a production inpainting pipeline no longer requires hosting a 12B-parameter model. Moebius delivers comparable quality on consumer hardware with meaningful inference headroom to spare.
