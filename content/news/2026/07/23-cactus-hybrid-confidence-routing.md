---
title: "The Model That Knows It Doesn't Know"
date: 2026-07-23T08:13:00+02:00
draft: false
slug: cactus-hybrid-confidence-probe-on-device-routing
categories: [inference]
tags: [on-device, confidence, routing, gemma, efficiency, mobile, calibration]
params:
  author: AI Beat Desk
  summary: >-
    Cactus Hybrid adds a confidence probe to Gemma 4 that reads internal activations
    to score each completion 0–1 and routes low-confidence queries to a cloud model.
    65–85% of queries stay on-device; overall accuracy matches Gemini 3.1 Flash-Lite.
    The probe generalizes to audio (0.79–0.88 AUROC) despite no audio training data.
---

Most hybrid AI deployments route between on-device and cloud models based on external signals — query length, topic category, user-configured settings, or a hardcoded list of task types. Cactus takes a different approach: the model decides for itself, based on what's happening inside its own forward pass, whether it's confident enough to answer locally.

[Cactus Hybrid](https://github.com/cactus-compute/cactus-hybrid) adds a lightweight confidence probe to Gemma 4's checkpoint that outputs a score between 0 and 1 for each completion. If the score exceeds a configurable threshold (default 0.85), the on-device model returns the answer. If not, `CompletionResult.needsCloudHandoff` flips to true and the application routes to a cloud model — Gemini 3.1 Flash-Lite in the reference implementation. The routing is fully automatic; there's no classifier over the query, no rule list, no user configuration required.

On their benchmarks, the result is that 65–85% of queries are handled entirely on-device, with 15–35% going to the cloud. Gemma 4 E2B Hybrid matches Gemini 3.1 Flash-Lite's overall benchmark performance despite being a substantially smaller model. The routing is doing real work: the system is correctly identifying which questions the small model handles well and which ones require more capacity.

What makes this more interesting than a standard cascading router is the probe's generalization. Cactus reports AUROC of 0.79–0.88 on audio benchmarks, despite the probe being trained entirely on text data. The probe isn't reading surface features of the query — it's reading something in Gemma 4's internal activations that correlates with whether the model has a reliable representation of whatever it's being asked, regardless of modality. If that finding holds on broader evaluation, it suggests the probe is measuring something closer to general model uncertainty than task-specific confidence.

The architecture is deliberately lightweight: a small probe over internal activations, added to the checkpoint with minimal overhead during inference. There are no ensembles, no Monte Carlo dropout passes, no temperature scaling — techniques that traditional uncertainty quantification relies on but that add latency and cost. The probe runs during the same forward pass that generates the completion, adding negligible compute.

The AUROC numbers (0.79–0.88) indicate reasonable but imperfect discrimination — good enough to reduce cloud calls substantially, not good enough to rely on blindly. Cactus exposes the confidence threshold as a configurable parameter, which is the right design choice: developers who care more about accuracy than cost can lower the threshold and route more queries to the cloud; privacy-sensitive applications can set local-only mode; cost-optimized deployments can raise the threshold and accept more risk on hard queries.

The integration story is practical — quickstarts for Cactus's own runtime, HuggingFace Transformers, llama.cpp, and MLX are all available in the repository. That covers most common on-device inference paths.

This comes out the same week as a [study showing](https://ai-beat.github.io/news/2026/07/ai-advice-accuracy-confidence/) that AI assistance makes humans more confident while making them less accurate — a calibration failure from the human side. Cactus Hybrid is attempting the opposite: a model that is conservative about its own confidence and routes to a stronger system when it doubts itself. Whether the confidence probe is well-calibrated in practice depends on the deployment distribution, but the framing is the right one. A model that knows when it doesn't know is more useful than one that answers everything at equal confidence — which is what most current small models do.

The broader primitive here — learned probes over internal activations used to gate expensive resources — seems broadly applicable. The same approach could extend to speculative decoding decisions, tool call triggers, or response caching. Small, cheap confidence heads trained on model internals are worth watching as an architectural pattern.
