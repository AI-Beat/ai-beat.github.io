---
title: "One Battery Charge"
date: 2026-10-08T06:11:32+00:00
draft: false
slug: smartphone-llm-finetuning
categories: [inference]
tags: [on-device, mobile, fine-tuning, mlx, apple]
params:
  author: AI Beat Desk
  summary: >-
    A new paper characterizes sustained fine-tuning of a 3B LLM on an iPhone 17 Pro: one battery charge is enough for a typical personalization run, and a broken MLX kernel they found and fixed cuts energy use by a third and speeds training 1.47x.
---

A new [arXiv paper](https://arxiv.org/abs/2610.06325) from Geyko, Mosbach, and Brinkmann measures something that's been claimed but rarely demonstrated at scale: sustained on-device fine-tuning of a multi-billion-parameter language model. They ran complete training loops on an iPhone 17 Pro — not a single step or a toy benchmark, but full personalization runs — tracking memory, step timing, heat, and energy throughout.

The headline result: a 3B-parameter model can be fine-tuned for a typical user in one battery charge, and the resulting LoRA adapters personalize about as well as adapters trained on a server. The phone works.

The less expected finding is thermal. Sustained training heats the device enough that throughput drops to roughly half its starting value. Pause-and-burst schedules didn't help. The throttling is persistent, and the authors don't expect software scheduling alone to fix it — the bottleneck is physics, not policy.

The most interesting part of the paper is a bug they found in Apple's [MLX](https://github.com/ml-explore/mlx). Most of each fine-tuning step is spent on the backward pass through the frozen base model. The base model's weights aren't being updated during LoRA fine-tuning, but gradients still need to flow through it to reach the adapter. MLX had a kernel written specifically for this backward pass that was never dispatched in practice, and produced incorrect results when it was. Nine of the ten other ML runtimes the authors evaluated don't accelerate this path at all; MLX had the right idea, but the implementation was broken and silently dead. After the fix — which is now merged upstream — adapter training runs 1.47x faster and uses about a third less energy.

A kernel that was written, never triggered, and silently wrong is exactly the kind of bug that accumulates in rapidly-developed ML runtimes. The authors' broader argument is architectural: most runtimes and operating systems treat training as a secondary or offline workload, scheduling resources accordingly. If on-device fine-tuning becomes a normal part of mobile AI, that assumption needs to change. Inference is clearly a first-class workload on modern phones; training is not, and the gap shows up in both throughput and energy efficiency.

The hardware is there. The remaining work is runtime support, thermal management, and making the throttling curve less steep. Personalized models that never leave the device — no server round-trips, no user data uploaded — are closer than they looked two years ago.
