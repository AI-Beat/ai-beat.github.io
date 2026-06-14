---
title: "Claude Passes an NMR Exam"
date: 2026-06-14T06:05:28+00:00
draft: false
slug: claude-nmr-chemistry
categories: [research]
tags: [research, science, chemistry, anthropic, multimodal]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic published a study showing Opus 4.7 matching or beating ChemDraw
    and MestReNova on 1D NMR spectroscopy tasks. The 80% J-coupling spacing
    accuracy — versus 26–35% for dedicated software — is the surprising number.
    The bidirectional structure elucidation capability has no direct equivalent
    in existing tools.
---

NMR spectroscopy sits at an awkward intersection of chemistry and pattern recognition. To determine a molecule's structure, a synthetic chemist collects a ¹H or ¹³C spectrum — a set of peaks at specific chemical shifts — and then reasons backward from peaks to structure. It is routinely time-consuming, requires training to do well, and is exactly the kind of task where software has historically helped at the forward direction (predict spectrum from structure) while leaving the inverse direction mostly to human expertise.

[Anthropic's research published today](https://www.anthropic.com/research/making-claude-a-chemist) tested Claude Opus 4.7 on both directions using established NMR datasets. The numbers are worth taking seriously.

On **forward prediction** — given a molecular structure, predict where ¹H and ¹³C peaks appear — Opus 4.7 achieved average absolute errors of ±0.079 ppm for proton shifts and ±1.37 ppm for carbon shifts. Those figures match or exceed ChemDraw and MestReNova, two pieces of software that chemists pay for specifically because they are good at this. The more striking result is on **multiplet prediction**: all three Claude models predicted J-coupling sub-peak spacings to within 0.5 Hz roughly 80% of the time, compared to 26–35% for ChemDraw and MestReNova. That gap is large. J-coupling constants depend on the through-bond geometry between neighboring hydrogen atoms — they capture structural connectivity in a way that shift positions alone do not — and predicting them accurately is harder. The fact that a general-purpose LLM outperforms specialized software here suggests the model is doing genuine structural reasoning rather than simple lookup from training data.

On **structure elucidation** — the inverse direction, from spectrum back to structure — Opus 4.7 recovered all eight simpler molecules correctly on every attempt. For the seven complex molecules (fused rings, spirocycles, with a starting-material hint provided), four were solved correctly on all three attempts; the remaining three were solved on two of three. Existing tools typically cannot perform this direction from 1D spectra without 2D NMR data, which requires additional experiments. Claude is doing it with 1D data alone.

The caveats are real. Anthropic designed this evaluation, and the test set sizes are small enough that variance matters — seven complex molecules is not a rigorous benchmark. The research should be read as "promising results suggesting this direction is worth pursuing" rather than "solved problem." Dedicated NMR software like MestReNova is also optimized for shift prediction, not multiplet patterns; that is a place where its design priorities show, and Claude's advantage may be specific to how the test was structured.

What makes this worth paying attention to is the category, not just the score. Chemistry tool access is one of those bottlenecks in lab-scale science: licenses are expensive, tools have steep learning curves, and interpretation requires expertise that not every lab member has. A model that can handle 1D NMR interpretation at quality parity with specialized software — and crucially can be asked follow-up questions, handle edge cases, and explain its reasoning — is genuinely useful in ways that existing software is not. The bidirectionality is the feature. Software can tell you what spectrum a molecule should produce. Claude can also look at a spectrum and suggest structures, and then discuss why.

Anthropic frames this as the beginning of a chemistry-focused science program, inviting research collaborators. The chemistry-specific prompt engineering and fine-tuning work that went into this evaluation is presumably ongoing. The immediate question is whether the structure elucidation results hold up on harder datasets with more diverse molecular scaffolds, and whether the approach works for ¹³C structure elucidation where coupling information is less available. The NMR case is a good argument for continued work. It is also a good example of how domain-specific evaluations — not generic benchmarks — are what actually tell you whether a model is useful for a real task.
