---
title: "Home-Trained and Jev-Accurate"
date: 2026-09-29T06:13:02+00:00
draft: false
slug: jeff-decision-model-calibration
categories: [inference]
tags: [inference, open-source, jev, classification, fine-tuning]
params:
  author: AI Beat Desk
  summary: >-
    Jeff is a set of Jev-compatible decision models — Qwen3.5 and Gemma 4
    fine-tunes — trained from scratch by a single developer in 2–3.5 hours on
    one RTX PRO 6000. The 2B variant hits 83.1% on five public benchmarks
    where Jev scores 83.0%, closing the quality gap that made every prior open
    replica a workaround rather than a replacement.
---

Ten days ago, writing about [the wave of open Jev replicas](/news/2026/09/jev-agent-decision-layer/), the conclusion was that calibration was the one thing the clones couldn't reproduce. Reading logits from a frozen model gets you a fast, cheap classifier; it doesn't get you a probability you can use as a gating signal. TypeSafe's hosted model returns a confidence score that actually means something. The open alternatives return a ranking.

[Jeff](https://github.com/firelex/jeff), which appeared on GitHub yesterday, is a different kind of attempt. It doesn't read logits from a frozen model. It fine-tunes from scratch.

The project is a set of decision models built on Qwen3.5 and Gemma 4, trained specifically for zero-shot classification using the [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) request format. The training pipeline generates synthetic data via Qwen3.8-Flash-Next running on a pair of DGX Sparks, filters it for quality, and applies full-weight fine-tuning with cross-entropy loss for one epoch — followed by temperature calibration to align the model's stated confidence with its actual accuracy. The 0.8B variant finishes in about 2 hours on a single RTX PRO 6000; the 2B variant in 3.5 hours.

The accuracy result is what stands out. Across five public benchmarks plus JevBench's hard tier, Jeff-Qwen3.5-2B scores 83.1%, edging Jev's 83.0% by a rounding error. The 0.8B lands at 79.1%, which is lower but still a meaningful improvement over the logit-reading replicas — and it runs at 22–28 ms on GPU, 60 ms on Apple M4 Max. The inference profile is correct for the use case: a single forward pass, calibrated probabilities over user-defined categories, no text generation, no autoregressive decode loop.

The temperature calibration step is what actually matters here. After the main fine-tune, Jeff fits a scalar temperature parameter on a held-out set so that the model's softmax outputs track its empirical accuracy. If it says 0.8 confidence, it should be right about 80% of the time. Without this step, the probabilities are arbitrary relative scores; with it, they're actionable gate thresholds. This is exactly what the previous generation of clones skipped, and it's what makes the confidence score usable in the pattern the Sep 19 post described — routing inside an agent harness, or gating autonomous tool execution by confidence level.

The [Sep 26 Ollaya post](/news/2026/09/jev-on-your-own-metal/) noted the quality gap honestly: local Laya scored 30.25 on a shared decision benchmark versus Jev's 63.29. Jeff is measuring on different benchmarks, so a direct comparison is imperfect, but the methodology gap is clear. Laya is a small model running without task-specific fine-tuning. Jeff is a fine-tune that treats zero-shot decision making as its sole training objective, with a calibration pass explicitly targeting the output property that makes the model useful for gating.

The practical implication is that the decision-model paradigm — small, fast, calibrated classifiers sitting in front of more expensive general-purpose models — is now reachable without a TypeSafe API key and without accepting a 2× quality penalty. Whether Jeff's calibration holds on harder distribution shifts is an open question; the test suite covers public benchmarks, and real production distributions are always messier. But the methodology (synthetic data generation, full-weight fine-tune, scalar temperature calibration) is now public and reproducible. Someone else can do this for a different domain in a weekend.

The thing to watch is whether TypeSafe responds. Their lead has been partly technical (calibrated training) and partly ecosystem (widespread SDK support, production integrations). Jeff closes the technical gap faster than anyone expected and is Jev API-compatible, which means it inherits the ecosystem. The argument for the hosted product narrows to calibration quality on difficult real-world distributions and the absence of infrastructure overhead — meaningful advantages, but thinner than they were a week ago.
