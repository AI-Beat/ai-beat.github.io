---
title: "VOID: Remove the Object, Rewrite the Physics"
date: 2026-04-05T09:00:00+02:00
draft: false
slug: netflix-void
tags: [video, open-source, generative-models]
params:
  author: AI Beat Desk
  summary: >-
    Netflix and INSAIT Sofia University released VOID, the first open-source
    video inpainting system that removes objects and regenerates the physical
    interactions they caused — not just the hole they left. It's Netflix's first
    public AI model release, built on a novel quadmask encoding and CogVideoX,
    under Apache 2.0.
---

Most video inpainting tools have a narrow job: cover the hole. Remove a person from a shot and you get a plausible background fill — the pixels are credible, but the scene is physically wrong. The shadow is gone but the light that cast it hasn't changed. The cup on the table moved but the water it displaced didn't. The system didn't understand why those things existed, only where.

[VOID](https://github.com/Netflix/void-model) (Video Object and Interaction Deletion), developed by Netflix Research and [INSAIT Sofia University](https://insait.ai) and released under Apache 2.0 on April 3, tries to solve a harder version of the problem: remove an object and regenerate the downstream physical consequences it caused. The [paper](https://arxiv.org/abs/2604.02296) is on arXiv.

The technical centerpiece is a new mask format called the **quadmask**: a four-value per-pixel encoding that distinguishes the primary object being removed, regions where it physically overlaps or interacts with other scene elements, the broader area affected by those interactions, and clean background. Standard inpainting masks are binary — in or out. The quadmask gives the model a richer description of which pixels need to change and *why*: "this pixel is part of the splash caused by the removed object, not just part of the removed object itself."

The base model is [CogVideoX-Fun-V1.5-5b-InP](https://github.com/aigc-apps/CogVideoX-Fun), Alibaba's video diffusion model, fine-tuned on synthetic training data from two sources: [Kubric](https://github.com/google-research/kubric) (Google's physics simulation environment, for object-only dynamics) and [HUMOTO](https://humoto.is.tue.mpg.de/) (Adobe/MPI's human-object interaction dataset rendered via Blender). The idea is that if you want a model to understand what happens when an object is removed, you train it on scenes where you know the ground truth of both states — which is easier to get synthetically than from real footage.

The inference pipeline runs in two passes. The first does the base inpainting using the quadmask conditioning. The second applies a warped-noise refinement step that improves temporal consistency — the main failure mode in video diffusion is flickering at object boundaries across frames. The system uses [SAM2](https://ai.meta.com/research/publications/sam-2-segment-anything-in-images-and-videos/) for segmentation and Gemini for reasoning about which interaction regions should be included in the mask, making it more automated than prior tools that required careful manual masking.

What makes this notable beyond the technical details is the context. Netflix has never publicly released an AI model before. The company sits on enormous proprietary video data and has obvious incentives to keep its production tooling internal. VOID is collaborative with an academic partner (INSAIT, a Bulgarian AI research institute with strong connections to ETH Zurich), open-weight, and commercially licensed — which signals either genuine commitment to open research or a calculated play to establish a standard before competitors do. Probably some of both.

The GPU requirements are not trivial (40GB+ VRAM, tested on A100), so this isn't running on consumer hardware. But the model weights, training code, and a Gradio demo are all public. For visual effects workflows that need to remove objects at scale without per-frame manual masking, this is a meaningful starting point.
