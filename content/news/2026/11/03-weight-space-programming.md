---
title: "An LLM Fixes Another LLM's Weights"
date: 2026-11-03T06:11:56+00:00
draft: false
slug: weight-space-programming
categories: [interpretability]
tags: [interpretability, model-editing, qwen, llm-tools]
params:
  author: AI Beat Desk
  summary: >-
    A blog post from this weekend describes using GPT-6 Astra to diagnose and
    edit the weights of Qwen3.5-0.8B without gradient descent, raising MATH-500
    accuracy from 7.9% to 48.1% by removing a single looping direction from two
    MLP layers. It's early and constrained, but the concept of an LLM doing
    targeted surgical weight editing is new territory.
---

There's a blog post from yesterday that's worth reading slowly: [Weight-Space Programming](https://rikiyat.github.io/blog/weight-space-programming/). The premise is straightforward — give a frontier LLM (GPT-6 Astra) Python access to another model's weights, a budget of forward passes, and a problem to fix — but the results are surprising enough that the caveats matter as much as the numbers.

The target model is [Qwen3.5-0.8B](https://huggingface.co/Qwen) in thinking mode, which has a documented flaw: it gets stuck in loops. Feed it a math problem and, more than 90% of the time, it generates reasoning tokens until it hits the context limit without ever producing an answer. On MATH-500 it scores 7.9%; on GSM8K, 13.8%. Both numbers mostly reflect timeout failures rather than wrong answers.

Astra was given practice problems from the GSM8K and MATH training sets and a budget of 50 forward passes to probe the weights and try edits. With only 10 or 25 passes, it made changes that barely moved the looping rate. With 50, it identified a direction that was being consistently written by the MLPs in layers 15 and 19 and which, when active, seemed to reinforce the loop. It subtracted that direction from 64 output columns in each of those layers — 131,072 weights, about 0.016% of the model — and the loop rate on MATH-500 dropped from 93% to 33%. MATH-500 accuracy went from 7.9% to 48.1%, GSM8K from 13.8% to 67.1%, IFEval from 21.0% to 62.3%.

What's notable is that the improvement isn't math-specific. IFEval measures instruction-following, not calculation, and it improved by the same magnitude. The authors take this as evidence that the fix removed a general looping obstacle rather than adding mathematical knowledge.

The control experiment is equally important. When Astra ran the same procedure on Gemma 3 270M — a model that doesn't loop — performance changed by about one point in either direction. No obvious obstacle, no meaningful edit. The authors say this suggests weight-space programming can remove things that block a model from using what it already knows, but can't install capabilities that aren't there.

There are real constraints here. The method required 50 forward passes, took too long, and the post is explicit that the harness is inefficient and context-limited. A separate experiment (Exp. 7) tried to use Astra to identify a sycophantic model from its weights alone, and it failed; it only succeeded after being allowed to inspect attention patterns on texts it had written. Editing weights from the outside, without gradient information and with a limited view of activations, has obvious limits.

But the conceptual shift is interesting regardless. Most model improvement today runs through some form of gradient descent: supervised fine-tuning, RLHF, DPO, continued pretraining. What this post explores is something different — an LLM doing interpretability work as a task, forming hypotheses about what specific weight subspaces compute, and then editing them directly. The researchers call it "weight-space programming" to distinguish it from training, and the label is accurate: it looks more like debugging than learning.

The closest prior work is mechanistic interpretability research — things like activation patching and circuit analysis — but those are typically human-driven workflows where a researcher writes code to probe and ablate. Here the LLM is doing the hypothesis generation and the editing. Whether that scales, or whether 50 forward passes is always going to be the ceiling for small models with obvious failure modes, is the open question. The authors say they're building a benchmark and a more rigorous harness. That's the right next step.
