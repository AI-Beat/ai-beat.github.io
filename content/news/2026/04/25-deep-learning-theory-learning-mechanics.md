---
title: "The Case for Learning Mechanics"
date: 2026-04-25T06:09:36+00:00
draft: false
slug: deep-learning-theory-learning-mechanics
categories: [research]
tags: [theory, deep-learning, scaling-laws, interpretability]
params:
  author: AI Beat Desk
  summary: >-
    Fourteen researchers across Berkeley, MIT, Harvard, and EPFL published a
    41-page manifesto arguing that a scientific theory of deep learning is not
    just desirable but already forming. They call it "learning mechanics" and
    point to five converging research threads — solvable models, tractable
    limits, empirical laws, hyperparameter theories, and universal behaviors —
    that together look something like what statistical mechanics looked like
    before it became statistical mechanics.
---

There's a familiar objection to deep learning theory: it works only for linear networks, or only in the infinite-width limit, or only under assumptions no real training run satisfies. The gap between what theorists can prove and what practitioners do is enormous.

A [position paper](https://arxiv.org/abs/2604.21691) by fourteen researchers — spanning Berkeley, MIT, Harvard, EPFL, and elsewhere — pushes back on that pessimism. Not by claiming the theory already exists, but by arguing the pieces are accumulating in a coherent direction. They call it **learning mechanics**.

The analogy is to classical mechanics. Physics studies how forces shape trajectories through space; learning mechanics would study how gradients shape trajectories through parameter space. The authors are explicit: "forces are mediated by fields; in deep learning, they are mediated by gradients." This isn't decorative language — it's a research program with specific content.

They identify five bodies of work that, taken together, look like a theory in formation:

**Solvable idealized models.** Deep linear networks can be solved exactly — Saxe et al. showed their training dynamics decouple into solvable ODEs under the right initialization. The neural tangent kernel (NTK) goes further: linearizing an arbitrary network around initialization yields exact predictions for generalization error in certain regimes. These aren't realistic models, but they make checkable predictions, and that's how science starts.

**Tractable limits.** As width goes to infinity, networks converge to either Gaussian processes (lazy/kernel regime) or richer feature-learning behavior depending on initialization scale. This isn't just intuition — it's a phase boundary, and it's derivable. Maximal Update Parameterization (µP), which emerged from this line of work, is now used at scale to transfer hyperparameters across model width without expensive re-sweeping.

**Simple empirical laws.** Neural scaling laws — test loss as a power function of compute, data, and model size — changed how frontier models are trained. The edge of stability, where loss sharpness reliably plateaus near \(2/\eta\) during training, appears across architectures. Neural collapse, where penultimate-layer representations converge to equiangular tight frames near convergence, has been replicated widely. These are macroscopic laws: they don't explain mechanism, but they constrain outcomes.

**Hyperparameter theories.** The linear scaling rule connecting learning rate to batch size has a derivation. µP tells you how the optimal learning rate depends on width — not what the best rate is, but how it scales. That's more useful than a point estimate.

**Universal behaviors.** Grokking, double descent, and similar representations appearing across architectures trained on similar data suggest some phenomena don't depend on microscopic details. The renormalization-group intuition from physics — macroscopic behavior is often independent of microscopic specifics — might actually apply here.

The authors propose seven desiderata for this theory, and the one that matters most is *humble*: it should know its own regime of applicability. They're not claiming learning mechanics will explain why your fine-tuned model forgets French. The theory is more like thermodynamics — aggregate properties of high-dimensional systems, not individual trajectories. The closest recent analogy is scaling laws: a simple empirical regularity powerful enough to change how trillion-dollar labs allocate compute. A broader theory covering training dynamics, representations, and generalization would be valuable even if it never explains everything.

There have been previous moments where deep learning theory seemed to be getting somewhere — the lottery ticket hypothesis, the NTK enthusiasm around 2019 — that didn't consolidate into general frameworks. The NTK turned out to describe a regime most real models don't train in. The difference now is that the empirical laws have already changed practice: scaling laws worked, µP works, neural collapse shows up. These aren't just mathematical exercises.

The paper is partly a coordination signal: fourteen researchers declaring that five research threads belong to a unified project, backed by a companion site at [learningmechanics.pub](https://learningmechanics.pub). Whether that framing sticks depends on uptake, but the underlying claim — that the evidence for a scientific theory of deep learning is stronger than the field's pessimism suggests — is defensible.

Worth at least the abstract and section headers if you work anywhere near ML theory.
