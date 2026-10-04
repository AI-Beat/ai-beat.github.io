---
title: "The Case for Translation-First Models"
date: 2026-10-04T08:11:39+02:00
draft: false
slug: bilibili-index-translate-150-languages
categories: [models]
tags: [translation, open-weights, moe, multilingual, bilibili, apache2]
params:
  author: AI Beat Desk
  summary: >-
    Bilibili's Index LLM Team released Index-Translate, a 35B MoE model family
    for 150 languages under Apache 2.0. Only 3B parameters activate per token.
    The family includes specialized checkpoints for speech translation,
    syllable-constrained dubbing, and full-document translation — and the
    MoE variant matches frontier generalist models on FLORES at a fraction of
    their inference cost.
---

The standard argument against translation-specific models goes roughly: frontier models can now translate well, and specialized models require separate maintenance and lack the generalist capabilities that make LLMs useful in the first place. That argument is true for casual translation. It breaks down for production workloads where you need consistent quality across 150 languages, low inference cost at scale, and open weights you can actually deploy.

[Bilibili's Index LLM Team released Index-Translate](https://huggingface.co/IndexTeam) on September 30 under Apache 2.0, and it's worth looking at as a counterargument to the "use a frontier model for everything" default.

## What the MoE design buys you

The flagship is the 35B-A3B-preview: 35 billion total parameters with only 3 billion active per forward pass. Mixture-of-experts has become the standard way to scale model capacity without scaling inference cost linearly, and translation benefits from it in a specific way. Different language pairs activate different expert subsets, so the model develops implicit specialization without needing separate per-language models. The routing happens at inference time with no additional latency overhead.

In practice, 3B active parameters at 35B total capacity means the inference cost sits closer to a dense 3B model than a dense 35B model. For a use case where you're processing documents across dozens of languages, that matters more than the raw parameter count.

The family also includes dense 2B and 9B variants for simpler deployments, and a set of specialized checkpoints that are more interesting than the headline numbers.

## The specialized variants

**Echo** handles speech-to-text and speech-to-speech translation — transcribing audio, then translating it to a target language. This is distinct from the text pipeline and presumably trained on paired speech data rather than adapted from the text models.

**Homura** is the most unusual: syllable-constrained dubbing via reinforcement learning. It generates translated text that matches the syllable count (and presumably rhythm) of the source audio, so the dubbing fits mouth movements without manual timing adjustments. This is a narrow capability but one that becomes critical when you're dubbing video content at scale across many languages. Getting it right through RL rather than heuristics suggests the team took the problem seriously.

**NativeLong** targets full-document translation with a 262,144 token context window (though they recommend staying under 32,768 in practice). Document-level context matters for translation quality in ways that sentence-level pipelines miss — terminology consistency, anaphora resolution across paragraphs, register maintenance throughout a long technical document.

## Performance

On FLORES, the 35B-A3B-preview scores 0.8794 COMET-22. The 9B dense model reportedly matches GPT-5.6-Sol and Gemini 3.5 Flash Lite on both WMT26 judge scores and FLORES. The off-target rate on low-resource language pairs is 2.4%, which the model card claims is the lowest in their comparison set. Low-resource is where translation quality typically degrades fastest — the limited parallel training data produces systematic errors and incorrect romanization — so that's the number worth scrutinizing most carefully.

The training pipeline used 167.77 billion multilingual tokens across constant and decay stages, followed by specialist fine-tuning with XCOMET-XXL as the RL reward signal and multi-teacher distillation. Using a translation-quality metric directly as the optimization target makes sense for this use case and is more principled than general RLHF preference data.

## When it matters

For most people building products, using a frontier API for translation is the right call — good enough quality, no maintenance burden, handles the occasional off-task request gracefully. Where Index-Translate becomes interesting is at the intersection of three constraints: cost sensitivity (3B active parameters vs. a frontier model's full pass), strict licensing requirements (Apache 2.0 is as permissive as it gets), and language breadth (150 languages with explicit low-resource support).

The Homura dubbing capability and NativeLong document pipeline suggest Bilibili is deploying this internally for video content and documentation workflows. Open-sourcing the result rather than keeping it proprietary is a useful contribution to the ecosystem, even if the primary motivation is likely recruiting and external validation. 150-language coverage at Apache 2.0 with GGUF quantized variants already on HuggingFace is a practical combination that will find use in production.
