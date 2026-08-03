---
title: "Train on Your Best Guess"
date: 2026-08-03T06:14:40+00:00
draft: false
slug: explorative-modeling-training-axis
categories: [research]
tags: [training, scaling, pretraining, diffusion, images]
params:
  author: AI Beat Desk
  summary: >-
    A new pretraining technique called Explorative Modeling adds a best-of-K
    selection step to the training loop — generate K candidates, keep the one
    closest to the target, backprop through only that one. The efficiency gains
    on image and video models are large and grow with scale, suggesting a genuine
    third axis alongside parameters and data. For autoregressive LLMs the gains
    are modest for now, but the underlying idea is worth watching.
---

Training a generative model is usually described as three steps: predict, compare to the target, update. The model follows the gradient of its own mistakes and gradually improves. This has worked well enough to produce every major model of the past decade.

What if you inserted a step before the update: generate K candidate predictions, score all of them against the target, keep only the best, then backprop through that one?

That is [Explorative Modeling](https://arxiv.org/abs/2607.27372) (XM), a paper from Alexi Gladstone submitted July 29. The method is almost disarmingly simple—the pseudocode is a handful of lines—but the efficiency numbers are not. On image generation, XM achieves 4.1× better FLOP efficiency and 6.2× better sample efficiency than standard training, and converges to FID 1.43 on ImageNet roughly 300× faster than a comparable baseline recipe. On robotics control tasks, single-step XM-trained models match diffusion performance while using 16–256× fewer inference steps.

The claim that elevates this beyond "useful trick" is the scaling signature. If exploration were a front-loaded optimization that padded early performance, you'd expect gains to flatten as the base model improves. Instead, they grow: the improvement from exploration goes from 13% to 23% as model size increases, and from 7% to 36% as data scales. That pattern—monotonic improvement with both axes—matches what you see with genuine scaling dimensions like parameter count and dataset size. Gladstone frames exploration as a "third pretraining axis," and the scaling curves make that framing defensible rather than just aspirational.

The mechanism is intuitive once stated. Standard training is in a sense pessimistic: it always trains on the model's first prediction, even when that prediction is nowhere near any reasonable answer. Adding exploration gives the model a chance to find a better path before committing to a gradient update. Critically, the K candidates are sampled from the model itself—no external oracle, no labeled preference data—so the training signal comes entirely from the original objective. It's a self-improvement loop baked into pretraining.

The honest caveat is the language model section. XM works well for continuous domains where generating a latent vector and comparing it to data is natural: images, video, robotics. For autoregressive text generation, the results are "modest." The paper attributes this to the difficulty of injecting a latent into autoregressive architectures, and suggests language models may be less bottlenecked by the specific problem XM addresses—generative expressivity, the tendency of models trained on averaged-out objectives to produce blurry or hedged outputs.

That caveat matters, because most people reading about a new pretraining technique care primarily about LLMs. A 4× efficiency gain on image generation is notable; "modest gains" on text is where the real question is. The paper is clear about this rather than burying it, which is worth something.

Still, the core idea has a kind of inevitability to it. Rejection sampling and best-of-N at inference have both proven productive—pick the best answer from a set of samples rather than trusting the first one. XM applies that same logic earlier, during training itself, which means the benefit compounds across the entire pretraining run rather than appearing only at serving time. Whether that unlocks a meaningful new axis for language specifically seems like an open and important question, and the robotics results suggest the technique will see serious follow-up work.
