---
title: "Images from a Field of Oscillators"
date: 2026-06-26T06:04:12Z
draft: false
slug: un-0-kuramoto-image-generation
categories: [research]
tags: [image-generation, architecture, physics, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Unconventional AI released Un-0, an image generator built not on diffusion
    or adversarial training but on Kuramoto coupled-oscillator dynamics. The
    learned parameters are coupling strengths between oscillators; the image
    emerges from a physical simulation rather than a stack of nonlinear layers.
    FID 6.74 on ImageNet-64 won't unseat SOTA, but the architecture is
    genuinely different and the code is MIT-licensed.
---

The standard recipe for image generation has been remarkably stable: either
a diffusion process that learns to reverse noise, or an adversarial game
between generator and discriminator, or more recently an autoregressive
model treating pixels as tokens. [Un-0](https://unconv.ai/blog/introducing-un-0-generating-images-with-coupled-oscillators/)
from Unconventional AI does something different — it generates images by
running a physics simulation and reading off the state.

The physics in question is the [Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model),
a classical description of coupled oscillators that has been used for
decades to study synchronization in everything from firefly flashing to
power grids. Each oscillator \(i\) carries a phase angle \(\theta_i\) that
evolves according to

\[
\dot{\theta}_i = \omega_i + \sum_j K_{ij} \sin(\theta_j - \theta_i)
\]

where \(\omega_i\) is the oscillator's natural frequency and \(K_{ij}\) is
the coupling strength to oscillator \(j\). In a standard Kuramoto analysis
these are usually taken as fixed or drawn from distributions. In Un-0 they
are the learned parameters — the entire representational capacity of the
model lives in the coupling matrix \(K\) and the frequency vector \(\omega\).

The generation procedure is: start every oscillator at a random phase,
inject the target class via a smaller set of "driver" oscillators that pull
the main population, run the dynamics forward to time \(T\), snapshot the
phase configuration, and pass that snapshot through a lightweight decoder.
The decoder is conventional — it uses a DINOv2 backbone — and accounts for
under 13% of total parameters. The oscillator layer does the heavy lifting,
and the ablations suggest the two components have a clear division of labor:
oscillators govern diversity, the decoder governs image quality.

What you get is a model with no diffusion schedule, no adversary, no
iterative denoising — just a physical system that you integrate forward
once. The largest released model has 322M parameters and reaches FID 6.74
on ImageNet-64. That puts it in early BigGAN and early iDDPM territory: not
a SOTA result by 2026 standards, but respectable for a first pass with an
entirely non-standard approach. CIFAR-10 models range from 1.3M to 19.4M
parameters (best FID: 8.86).

The efficiency argument is more speculative. The blog post mentions the
~1000x energy claim, which comes from the theoretical possibility of
implementing Kuramoto dynamics in analog hardware, where physical oscillators
synchronize at essentially zero computational cost. That is real physics —
analog oscillator networks are genuinely cheap to run once built — but the
current PyTorch implementation obviously does not realize that advantage;
it simulates the differential equations on GPU like anything else. Whether
the architecture is a proof of concept for future neuromorphic or analog
hardware, or just an interesting mathematical curiosity, is an open question.

The code is [on GitHub](https://github.com/unconv-ai/un-0) under MIT
license with six pretrained checkpoints hosted on Hugging Face. The
implementation is plain PyTorch with separate training pipelines for
CIFAR-10 and ImageNet-64. It is reproducible and small enough to train on
a single GPU for CIFAR-10, which means it is actually possible to experiment
with the architecture rather than just observe it from a distance.

What is interesting here is not that Un-0 beats diffusion at ImageNet — it
does not. It is that the representational substrate is entirely different.
Most architecture exploration in image generation is about adjusting the
diffusion process (deterministic samplers, flow matching, consistency models)
or swapping in different backbone networks. Un-0 asks whether you need
backpropagation through a stack of layers at all, or whether you can instead
learn the parameters of a dynamical system and let physics do the forward
pass. That is a question worth asking, even if the current answer is "FID
6.74."
