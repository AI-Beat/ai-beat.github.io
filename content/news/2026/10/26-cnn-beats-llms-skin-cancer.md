---
title: "The Specialist's Edge"
date: 2026-10-26T06:13:46+00:00
draft: false
slug: cnn-beats-llms-skin-cancer
categories: [research]
tags: [medical-ai, benchmarks, computer-vision, llm-limits]
params:
  author: AI Beat Desk
  summary: >-
    A comparison of fine-tuned Inception v3 against GPT-6 Astra and Claude
    Fable 5.1 on skin lesion diagnosis finds the decade-old CNN dramatically
    outperforms both frontier models on overall accuracy — 62% vs 42% — by
    being purpose-built for exactly this task. The result is a useful reminder
    that frontier capability and specialist capability remain different things.
---

A [study published yesterday](https://lynchlab.net/llm-skin-cancer) by dermatologist Magnus Lynch pits two frontier multimodal LLMs — GPT-6 Astra and Claude Fable 5.1 — against a fine-tuned Inception v3 on classifying 200 biopsy-proven skin lesions from the MILK10k public dataset. The CNN is a decade-old architecture, trained specifically for this task. The LLMs are the best general-purpose vision models available. The score: CNN 62%, LLMs 42%.

That gap is larger than most people would guess, and it's worth understanding why.

The CNN's advantage is almost entirely on the easy cases. For the 100 benign lesions in the dataset — moles and seborrhoeic keratoses — the CNN recognised 63 correctly, against 54 and 58 for the two LLMs. At a discharge threshold the CNN was comfortable enough to send 76 of the 100 safe lesions home without biopsy; the LLMs managed 22 and 32. That's a large difference in clinical utility, if you trust the model enough to use it for triage.

The melanoma numbers tell a different story. The CNN found 13 of 30 melanomas; the LLMs found 8 and 11. But here's the caveat Lynch himself buries in the small print: when the CNN discharged, it discharged 18 of those 30 melanomas. The LLMs discharged only 3 and 4. The CNN is faster and cheaper at calling harmless lesions harmless. It is also more willing to wave dangerous ones through.

That tradeoff is the interesting thing. A model optimised for sensitivity (catch every melanoma) and a model optimised for specificity (don't biopsy things that don't need biopsying) will look very different on this benchmark, and neither is straightforwardly "better" without a clinical context that tells you which errors are acceptable. Lynch's framing leans toward the CNN as the winner — its top-1 accuracy is higher, its discharge of safe lesions is much better — but the melanoma miss rate is the number that a clinical deployment would care about most.

The broader point stands even with that caveat. Inception v3 was fine-tuned on exactly this kind of data — dermatoscopic images of skin lesions — and has been trained to distinguish these six specific categories. GPT-6 Astra and Claude Fable 5.1 were trained on everything, including medical images, but without any specific focus on dermatoscopy. The fine-tuned model is doing pattern matching in a feature space it learned precisely for this task. The LLMs are probably reasoning in a more language-like way about visual features, which is flexible but apparently less precise here.

This is not surprising in principle. Task-specific fine-tuning has always been expected to win on the task it was fine-tuned for. What makes the result interesting in 2026 is the size of the gap and what it implies about the current narrative. For the past two years the argument has been that frontier LLMs are becoming good enough at specialized tasks that dedicated models are hard to justify — you can route everything through a general-purpose API, avoid the maintenance overhead of separate models, and accept a small accuracy hit. A 20-percentage-point accuracy gap is not a small hit. For clinical decisions with real consequences, it matters.

There are real limitations to this study. Two hundred lesions is a small sample. Biopsy-proven lesions skew toward ambiguous, harder cases — a dermatoscope image that was obviously benign in clinic never made it to the pathologist, so the dataset underrepresents the easy end of the distribution where LLMs might do better. And the "decade-old CNN" framing somewhat misleads: a well-tuned Inception v3 fine-tuned on a purpose-built dataset is not an obsolete system; it is a simple system that was given exactly the training signal it needed. Lynch calls it a "basic baseline" but that's modesty, or context: it's basic in architecture, not in applicability.

What the study does is add one more data point to the growing literature on where general-purpose LLMs fall short of specialized alternatives. LLMs are genuinely useful across a wide range of tasks. They are not uniformly the best tool for every subtask within that range. The case for building or keeping a specialized model is still worth making, task by task, with actual numbers — and studies like this one are a good reason to run those comparisons before assuming the general-purpose model is good enough.
