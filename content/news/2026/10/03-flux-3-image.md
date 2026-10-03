---
title: "Laying Out an Image Like a Document"
date: 2026-10-03T08:11:44+02:00
draft: false
slug: flux-3-image-bounding-boxes
categories: [tools]
tags: [image-generation, diffusion, black-forest-labs, flux, open-weights]
params:
  author: AI Beat Desk
  summary: >-
    Black Forest Labs ships FLUX 3 Image with bounding-box composition, multi-step
    targeted editing, and native 4K output — bringing structured spatial control
    to diffusion that was previously only available through ControlNet-style
    workarounds. Open weights announced for coming weeks.
---

The core frustration with prompt-driven image generation has always been placement. You can describe a scene in detail, but you can't say "put the person on the left third of the frame, the building in the upper right, and keep everything else." You get close. You iterate. You use inpainting to fix the parts that drifted. With complex compositions, this becomes a multi-hour workflow.

[FLUX 3 Image](https://bfl.ai/models/flux-3-image), released by Black Forest Labs on October 1, takes a different approach. The model accepts bounding boxes as first-class inputs — normalized 0-1000 coordinates defining where each element should appear — and generates accordingly. You describe the subject, provide its box, optionally provide reference images, and the model places it where you asked.

The canvas system is clean: `[y_min, x_min, y_max, x_max]` on a 0–1000 grid, aspect-ratio independent. Up to ten reference images can anchor element appearance. The rest of the scene fills in around the specified constraints.

## What selective editing actually means

The other piece worth attention is how edits work. Most generation-based editing approaches have a leakage problem: asking the model to change one region tends to drift the rest. You fix the shirt color and the background shifts slightly. You swap a face and the lighting changes.

FLUX 3 Image makes precise editing its explicit design target. The pitch is that you can replace an element, reposition it, or re-describe its appearance without touching the areas you want to preserve. Multiple sequential edits accumulate without degrading the untouched regions. Whether this holds up in practice across difficult cases — occlusion, lighting-dependent subjects, high-frequency textures — is something users will stress-test quickly.

The bounding box approach also makes this more tractable architecturally. When the model knows which region is being changed, it has a cleaner signal about what it should and shouldn't modify. This is a different design philosophy than pure diffusion editing, which treats the whole image as the canvas and relies on masking to protect regions after the fact.

## Context: the FLUX 3 family

FLUX 3 Image is the third component of a larger release. Black Forest Labs announced the multimodal FLUX 3 system in July 2026 with video and action prediction (robotics) components; the image side was withheld for later. The underlying claim is that all three modalities — static images, video with synchronized audio, robotic action prediction — come from a single jointly-trained model. The image model ships now as a paid API with a 50% launch discount through October 8, with commercial weight licensing for on-premise deployment, and an open-weight release described as coming within weeks.

The 4K native output is the other headline number. Generating directly at high resolution rather than upscaling avoids the artifacts that show up when you scale a lower-res generation, particularly around fine detail like hair and text. Whether the model's coherence at 4K matches its quality at standard resolutions is the thing to check.

The open-weight timeline is vague — "coming weeks" — but it's a firm commitment rather than an aspiration, which puts FLUX 3 Image on a different track from proprietary-only image APIs. For workflows that need local generation or fine-tuning on private datasets, that matters more than the API pricing.

The structured spatial control here isn't new as an idea — ControlNet demonstrated the demand for it years ago — but embedding it natively into a frontier image model rather than bolting it on changes the ergonomics considerably. It's worth watching how the community actually uses the bounding-box primitives once the open weights land.
