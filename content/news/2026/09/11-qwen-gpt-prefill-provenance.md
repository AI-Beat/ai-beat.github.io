---
title: "A Ghost in the Reasoning"
date: 2026-09-11T06:08:15+00:00
draft: false
slug: qwen-gpt-prefill-provenance
categories: [training]
tags: [training, qwen, data-provenance, reasoning, contamination]
params:
  author: AI Beat Desk
  summary: >-
    A researcher prefilled Qwen3.8 A95B with the first 1% of a GPT-5.5 Pro
    chain-of-thought and measured how much the model continued following it.
    The resulting +18 pp jump in overlap — much larger than seen in other models
    — is a practical signal that Qwen's post-training data may include GPT-5.5
    Pro reasoning traces, and points to a general method for probing where a
    model learned to think.
---

A researcher [posted a short experiment](https://gist.github.com/wsxiaoys/e0286dc6bb624ff5fdf49e7f4c528ba3) this week with a clean design and an uncomfortable implication. The setup: take the first 1% of GPT-5.5 Pro's reasoning trace on a given problem, insert it as a prefill into other models, and measure how much those models continue along the same reasoning path. The metric is lexical overlap between the model's first 100 output tokens and GPT-5.5 Pro's full reasoning.

For most models, the prefill moves the needle modestly. Qwen3.8 A95B was different. Without any prefill, it already achieves 16.79% overlap with GPT-5.5 Pro's reasoning — higher than you'd expect from coincidence but not implausible given that frontier models tend to converge on similar approaches to common problem types. With the GPT-5.5 Pro prefill inserted, that number jumps to 34.97%: a +18.18 percentage point increase. The STEM category shows the sharpest response (+26.99 pp), with non-STEM and puzzle problems not far behind (+12.80 pp and +14.75 pp respectively).

The interpretation offered is cautious but direct: "Qwen may have learned from GPT-5.5 Pro, or from a closely related GPT model, rather than from Opus." What the prefill is doing, in this reading, is activating a latent pattern already present in Qwen's weights — a pattern that looks like GPT-5.5 Pro's chain-of-thought because the model was trained on examples of it. Other models that don't show the same response either learned from different sources or learned reasoning patterns that are less susceptible to this kind of priming.

This method is a probe, not proof. High overlap after a prefill shows the model is capable of following a particular reasoning style; it doesn't rule out the possibility that the model would follow *any* sufficiently coherent prefill in a similar way. You'd want to test with other proprietary models' reasoning traces to establish specificity — does Qwen respond this strongly to a Claude 4.8 prefill? To Gemini 3.8's? If the response is GPT-5.5-Pro-specific rather than a general willingness to follow a well-formed chain-of-thought, the training data inference becomes much stronger.

The broader context here is that large-scale post-training with synthetic data is now routine, and the synthetic data often comes from proprietary models. This is widely assumed to be happening — several open-weight models have improved dramatically in the past two years in ways that track closely with proprietary model capabilities — but it's rarely documented in model cards, and "distillation" appears in technical reports with varying degrees of specificity. What this experiment offers is a practical forensic technique: probe the model's priors by testing how readily it continues reasoning it might have been trained on.

The asymmetry is worth sitting with. A model's training data influences which reasoning patterns feel natural to it, which means the provenance of synthetic training data becomes a property of the model's reasoning style, not just its benchmark scores. If Qwen's STEM reasoning looks like GPT-5.5 Pro's because it was trained on GPT-5.5 Pro traces, that's not necessarily a problem for users — they get high-quality reasoning either way — but it does mean that claims about model independence, capability origin, and the diversity of the frontier model ecosystem deserve more scrutiny than they usually get.

The experiment also applies in reverse. Kimi K3 — the model Cognition used as the base for SWE-2, [released the same week](https://cognition.com/blog/swe-2) — reportedly shows a similar response to Claude 4.8 prefills, suggesting Moonshot AI's post-training incorporated Claude-family reasoning. If that pattern holds across multiple open-weight models, then the "open-weight" ecosystem may be less independent of proprietary development than it appears on model cards.
