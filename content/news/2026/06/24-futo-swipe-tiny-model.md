---
title: "2.5 Million Parameters Beats Gboard"
date: 2026-06-24T07:30:00+00:00
draft: false
slug: futo-swipe-tiny-model
categories: [open-source]
tags: [open-source, inference, on-device, mobile, privacy]
params:
  author: AI Beat Desk
  summary: >-
    FUTO released the models behind their swipe keyboard — a three-component
    stack totalling 2.5 million parameters that achieves 26% fewer errors than
    Gboard on their benchmark. It trains on one workstation GPU, runs on
    low-end Android devices in milliseconds, and is the first freely licensed
    open swipe-typing model. It's a reminder that model scale is a tool, not
    an objective.
---

In a week dominated by 35B and 397B parameter models, [FUTO Swipe](https://swipe.futo.tech/) arrived on the scene at 2.5 million parameters and beat Google's Gboard by 26% on word error rate.

FUTO Keyboard has been around for a while — it's an offline, open-source Android keyboard that doesn't phone home. The autocorrect was already reasonably good. What was missing was a competitive swipe typing implementation: the gesture-based input mode where you trace a path across the keyboard without lifting your finger. The good swipe implementations — Gboard's in particular — are proprietary, and the open-source alternatives have historically been mediocre. FUTO spent over a year building [a replacement](https://swipe.futo.tech/).

The architecture is deliberately split into three small models rather than one. An Encoder (635K parameters) handles general prediction: it's layout-agnostic and language-agnostic, generating candidate words from the gesture path. A ContextLM (1.5M parameters) is a tiny language model that re-ranks those candidates using context — it knows what words appeared previously and filters out word choices that would be linguistically nonsensical in context. A Decoder (304K parameters, language and layout-specific) provides high-accuracy decoding for particular keyboard layouts, currently QWERTY English. The total comes to about 2.5M parameters, runs in milliseconds on low-end hardware, and was trained on a single workstation GPU.

The 26% error reduction over Gboard comes from their internal benchmark, so the standard caveats apply about benchmark construction and selection effects. But the result is directionally plausible: swipe typing is a constrained sequence-matching problem, not open-ended generation. The gesture path provides very strong signal about intended words, and a small model that's specifically trained for this task can do quite well. You don't need GPT-scale knowledge to figure out that someone tracing a path from G to R to E to A to T probably meant "great."

What's notable here is what FUTO chose *not* to do. They didn't take a large language model and fine-tune it on keyboard data. They didn't try to build a general model that handles both swipe and voice and autocorrect in one system. They built three small, purposeful models and composed them. The ContextLM is tiny (1.5M params) because it doesn't need to understand the world — it just needs to understand which word makes sense after the previous one, within a swipe-typing context where the candidate set is already narrowed.

The licensing is mostly open. The C++ inference library is GPL, the dataset is freely available on HuggingFace, and the models use FUTO's own model license which requires visible user attribution in apps that use them — not fully permissive, but serviceable for open-source projects. They describe this as "the first useful free and open Swipe model," which seems accurate; the alternatives before this were either proprietary or demonstrably worse.

There's a broader point here worth stating plainly. The field has a tendency to treat model scale as a proxy for model quality, which works surprisingly often for general tasks but breaks down for specialized ones. Swipe typing is one of those specialized tasks where the problem is well-defined, the signal is strong, and the right architecture is a purpose-built small model rather than a shrunken general one. The team trained this on a single GPU; the environmental and computational costs are negligible compared to any frontier model. And it apparently works well.

If you're running an Android device and want offline-first, privacy-preserving keyboard input, [FUTO Keyboard](https://keyboard.futo.org/) with the Swipe update (version 0.1.29, released June 13) is a meaningful practical improvement. If you're building something that needs gesture-based text input and can't use Gboard, the models are on HuggingFace and the inference code is open. The benchmark methodology is on the project page if you want to evaluate it against your own dataset.
