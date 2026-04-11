---
title: "No Teacher Required"
date: 2026-04-04T19:05:00+02:00
draft: false
slug: self-distillation-code
tags: [training, llm, code]
categories: [research]
params:
  author: AI Beat Desk
  summary: >-
    A new arXiv paper shows that sampling a model at high temperature, filtering
    outputs that actually run, and SFT-ing on the result lifts Qwen3-30B from
    42.4% to 55.3% on LiveCodeBench — no reward model, no external verifier, no
    teacher model needed.
---

The standard recipe for improving a model at code generation involves some combination of a verifier (run the tests), a stronger teacher model (distill down), or reinforcement learning with human feedback. A [paper posted to arXiv on April 1](https://arxiv.org/abs/2604.01193) removes all three. The method is called simple self-distillation (SSD), and the paper's title — "Embarrassingly Simple Self-Distillation Improves Code Generation" — is not underselling it.

The procedure has three steps: sample many solutions from the model using temperature 0.8 and top-p 0.95; keep only those that parse and execute without errors; fine-tune the original model on that filtered dataset with standard supervised learning. No reward model. No comparison to a reference solution. No RL training loop. Just "did this code run?" as the only signal.

The results on [LiveCodeBench v6](https://livecodebench.github.io/) are meaningful. Qwen3-30B-Instruct goes from 42.4% to 55.3% pass@1 — a 13-point gain, concentrated on the harder problems. The method generalizes: the authors tested Qwen and Llama models at 4B, 8B, and 30B parameters, in both instruct and thinking variants, and saw consistent improvements across the board.

The explanation the paper offers is interesting. They frame standard LLM decoding as suffering from a "precision-exploration conflict." When a model generates code, its token distributions mix two populations: tokens where the correct answer is fairly sharp (you need `int`, not `str`) and tokens where several paths are equally valid (the algorithm could be iterative or recursive). Beam search and greedy decoding collapse both to the mode, cutting off useful variation. High-temperature sampling opens the distribution back up, but now you're sampling noise along with the diversity. SSD is claimed to resolve this: the fine-tuning step reshapes the distribution in a context-sensitive way, suppressing "distractor tails" in high-precision regions while preserving spread where spread is useful.

Whether that explanation is mechanistically accurate is harder to verify, but the empirical result is there. The fact that just filtering by execution success — no semantic correctness check, no test suite — is enough to produce useful training signal says something about the density of the existing model's output space. The model, at temperature 0.8, is already generating plausibly correct solutions at a high enough rate that a simple filter recovers a useful dataset. This probably doesn't work as well for a weaker base model or a harder task distribution where correct solutions are rare.

The "no external teacher" angle is practically relevant for anyone who wants to improve a model but can't run a stronger teacher at inference time or build a test harness. If you have a model and a dataset of problems with runnable-code as the output, you can apply SSD. The compute cost is roughly 2× inference for the sampling pass plus one fine-tuning run, which is manageable at the 4B–30B scale the paper covers.

It also fits into a broader pattern of "the model's own distribution contains more signal than we're extracting." Techniques like chain-of-thought, self-consistency, and process reward models all exploit the model's output space in different ways. SSD does it with the simplest possible signal — syntax and runtime — and gets surprisingly far. The paper is worth reading if you're building code evaluation pipelines or thinking about cheap paths to capability gains on a deployed model.
