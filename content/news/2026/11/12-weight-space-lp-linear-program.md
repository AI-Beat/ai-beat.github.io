---
title: "The LLM Was Solving an LP"
date: 2026-11-12T06:16:56+00:00
draft: false
slug: weight-space-lp-linear-program
categories: [interpretability]
tags: [interpretability, model-editing, llm-tools, quantization]
params:
  author: AI Beat Desk
  summary: >-
    Nine days after we noted GPT-6 Astra fixing Qwen3.5-0.8B's looping problem
    via weight editing, Nathan Barry shows that the LLM's eight-hour trial-and-error
    search is equivalent to a linear program that solves in a minute and touches 8×
    fewer weights. The original result stands, but the mechanism is a lot simpler than
    it looked.
---

[Earlier this month](https://ai-beat.github.io/news/2026/11/weight-space-programming/) we looked at a result from MIT CSAIL: GPT-6 Astra, given Python access to Qwen3.5-0.8B's weights and a budget of forward passes, found and removed the direction in two MLP layers that was causing the model to loop. MATH-500 accuracy went from 7.9% to 48.1%. The framing was "weight-space programming" — an LLM doing interpretability work as a task.

[Nathan Barry's follow-up](https://nathan.rs/posts/weight-space-lp/) from November 9 complicates that framing in an interesting way. His claim: Astra wasn't really doing anything novel. It was solving a linear program through trial and error, and you can just solve the LP directly.

The setup mirrors Astra's. You have a "loop direction" — a vector in weight space that, when active, reinforces the loop — and you want to subtract it from some set of weight columns. Astra searched for which columns to touch and by how much, spending 28 forward passes across about eight hours before settling on 130,188 weight changes. Barry reformulates this as an LP: minimize the \(L_1\) norm of changes (seeking sparsity) subject to constraints that push the looping tokens away from the loop direction while leaving productive tokens mostly untouched. Two LP solves, running in roughly a minute, pick 16,291 weight changes — about 8× fewer than Astra. Accuracy on GSM8K, MATH-500, and IFEval comes out within noise of the original.

The really deflating finding is the steering-vector experiment. Adding a constant push along the loop direction at inference time recovers essentially the full fix without touching weights at all. This suggests the edit was mostly a constant shift: the same offset applied to every token regardless of context. The per-token constraints in the LP turned out not to matter much, because the looping behavior didn't require fine-grained steering — it just needed a consistent nudge away from one attractor. As a further test of how much the LLM direction mattered, a heuristic loop direction (derived without calling an LLM) plugged into the same LP setup gets about 93% of the way to Astra's accuracy numbers, with 16,271 weight changes instead of 16,291.

This doesn't undermine the original result — the edit works, the numbers are real. But it does reframe what Astra was doing. The LLM spent eight hours discovering something a constrained optimization problem could recover in sixty seconds, and the "surgical" weight editing turns out to be nearly equivalent to a fixed runtime intervention. For the specific failure mode being fixed (a looping attractor), direct optimization beats LLM-guided search by a large margin on efficiency.

Barry is careful about scope. The LP still relies on Astra to identify the right layers and the loop direction; it replaces the column-selection search, not the whole pipeline. And loops are a particularly clean target — a single spurious direction, a clear failure mode, a relatively simple constraint set. Behaviors that require genuinely token-selective changes (fine-grained refusals, context-dependent style shifts) would likely need something more expressive than a constant subtraction.

The broader takeaway is that weight-space programming may turn out to be a spectrum. At one end, you have pattern-matching edits that are really just offline steering vectors baked into weights — solvable with standard optimization, cheap and interpretable. At the other end, you presumably have edits that require the kind of adaptive, hypothesis-driven search that an LLM can provide. The interesting research question is figuring out where on that spectrum any given editing task falls before committing to an expensive LLM-driven search.
