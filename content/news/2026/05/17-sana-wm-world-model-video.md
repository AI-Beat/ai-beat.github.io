---
title: "One Minute of 720p World on One GPU"
date: 2026-05-17T06:12:00+00:00
draft: false
slug: sana-wm-world-model-video
categories: [research]
tags: [video, world-model, diffusion, nvidia, open-source, inference, camera-control, linear-attention]
params:
  author: AI Beat Desk
  summary: >-
    NVIDIA's SANA-WM generates 60-second, 720p video from a single image and a
    camera trajectory — on a single GPU. The open-source 2.6B-parameter model
    achieves 36× higher throughput than prior open-source world models and
    ships under Apache 2.0.
---

World models for video have been one of those capabilities that everyone agrees
matters — autonomous vehicles, robotics simulation, game engine replacement —
but where the compute and memory requirements have kept serious open-source work
firmly in the "nice demo, impractical to run" tier. NVIDIA's [SANA-WM](https://arxiv.org/abs/2605.15178),
released this week, is a notable step out of that tier.

The model is a 2.6B-parameter Diffusion Transformer that takes a single image
and a 6-DoF camera trajectory as input and produces a 720p, 60-second video
that follows the trajectory faithfully. What's notable isn't just the resolution
or the duration (prior open-source world models maxed out well below both) but
the inference cost: a distilled variant generates a full 60-second clip in 34
seconds on a single RTX 5090 with NVFP4 quantization. Training ran on 64 H100s
for 15 days using around 213,000 public video clips — meaningful but not
hyperscale. The model is [Apache 2.0 licensed](https://github.com/NVlabs/Sana)
with weights and code available.

The throughput number — 36× higher than prior open-source baselines — is
striking, and it comes primarily from the architecture choice. SANA-WM uses a
hybrid linear attention design it calls Gated DeltaNet (GDN): within each
frame, it applies the efficient delta-rule linear attention from the DeltaNet
line of work; across frames it layers in standard softmax attention to handle
long-range temporal dependencies without blowing up memory. The frame-wise
linear attention means that generating longer sequences doesn't hit the
quadratic wall at the intra-frame level, and the two-stage pipeline — a
coarse generation pass followed by a refinement stage — improves temporal
consistency without requiring a separately trained upsampler.

Camera control is handled by a dual-branch design: a control branch that
processes the camera trajectory signal runs in parallel with the main
generation branch, then injects the camera conditioning through cross-attention
at each transformer block. The paper reports strong 6-DoF trajectory following
on standard driving benchmarks. The annotation pipeline that extracts camera
poses from public videos — allowing training without proprietary labeled data —
is probably the part that gets underappreciated in coverage of this kind of work.
Having a reproducible recipe for building your own training set is genuinely
useful.

What this practically enables: researchers building simulation environments for
robotics or autonomous systems no longer need a closed, proprietary video
foundation model to get minute-scale, controlled video generation. The single-GPU
inference cost puts it within reach of anyone with a recent high-end GPU.

The remaining limitations are real. SANA-WM is designed for forward-facing
camera trajectories in the driving/outdoor domain; the training data distribution
reflects that. It doesn't handle arbitrary scene composition or indoor environments
with the same fidelity. And 34 seconds on an RTX 5090 is still 34 seconds —
real-time generation of 720p minute-scale video is not on the table here. But as
a research artifact that's actually runnable on one GPU, ships under a permissive
license, and closes most of the quality gap with closed-source alternatives, it
earns the attention it's getting.
