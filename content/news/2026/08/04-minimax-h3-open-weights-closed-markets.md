---
title: "Open Weights, Closed Markets"
date: 2026-08-04T06:14:02+00:00
draft: false
slug: minimax-h3-open-weights-closed-markets
categories: [models]
tags: [video-generation, open-source, licensing, copyright, china]
params:
  author: AI Beat Desk
  summary: >-
    MiniMax released open weights for H3 on August 3 — a 33B video generation
    model that produces native stereo audio in the same forward pass as video
    and currently tops open-weight video rankings. The catch is a geographic
    restriction that effectively prohibits use in the US, EU, UK, and South
    Korea, the result of active copyright litigation from Hollywood studios and
    regulatory uncertainty in Western markets.
---

[MiniMax H3](https://huggingface.co/MiniMaxAI/MiniMax-H3) is a good video model. Released to Hugging Face on August 3 — after a week as API-only — it's currently ranked first among open-weight video models by third-party evaluations, and competitive with closed alternatives on several dimensions. The 33B parameter model accepts text, images, existing video, and audio as inputs, generates clips up to 15 seconds at 2K resolution, and does something few video models do: audio is produced natively in the same forward pass as video, in stereo, rather than synthesized separately and merged afterward. The distinction matters because post-processed audio always sounds slightly detached; native audio lets the model learn correlations between what's visible and what's audible rather than having to infer them after the fact.

The [ComfyUI team shipped day-zero support](https://blog.comfy.org/minimax-h3-day-0-support-in-comfyui/) with meaningful efficiency work: they pruned modulation weights into lookup tables, applied int8 quantization, and wrote custom kernels, dropping the model from 123.6 GB full precision to 42.5 GB. An RTX 3060 is reportedly sufficient for local inference with those optimizations applied.

So far, so good. Then you read the license.

The [MiniMax H3 Community License](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/docs/QA-about-License.md) prohibits use, distribution, modification, and hosting — and also prohibits using the model's outputs — in the United States, European Union, United Kingdom, and South Korea. This covers the overwhelming majority of likely open-weight users.

The restrictions aren't arbitrary. A [US court denied MiniMax's motion to dismiss](https://www.kucoin.com/news/flash/minimax-restricts-h3-license-in-u-s-eu-uk-and-south-korea-due-to-hollywood-copyright-lawsuit) in May, in a case brought by Disney, Universal, and Warner Bros. Discovery over training data and generated outputs. The EU AI Act, UK rules, and South Korean regulations add further uncertainty that MiniMax isn't willing to absorb right now. Video models face sharper legal exposure than text models: generated video is more visually similar to training material, the use cases overlap more directly with protected creative work, and courts are still working out whether existing fair use and copyright doctrine applies.

MiniMax's stated position is that the exclusions mean "not yet, not never" — US users can apply for authorization and commit to a content compliance mechanism, and the company says it expects to expand geographic availability as the legal picture clarifies. That may be true, but it's worth being clear about what "open weights" means in this context: you can download the checkpoint, provided you're not in most of the countries where AI development is concentrated.

The situation highlights a pressure point for Chinese AI labs releasing models internationally. Open-sourcing weights has been an effective strategy for model adoption — Kimi K3 and GLM-5.2 both benefit from being freely runnable — but video is different. Text generation sits at a distance from the training data. Video generation can produce frames that look like specific scenes from specific films, which is a harder legal position to defend under existing copyright frameworks in the US and EU.

The model is technically strong enough that the restriction will frustrate people. It's also a preview of what model licensing looks like when copyright exposure is sufficiently concrete: geographic carve-outs rather than use restrictions, because the liability runs to the territory's legal system rather than to any specific application. For a researcher or developer in Canada, Australia, or Japan, H3 appears freely usable today. For anyone working in Western Europe or the US, it's a model you can read about but probably shouldn't run.

Whether the weights are "open" in any meaningful sense when they're restricted in most jurisdictions is a definitional question worth sitting with. What's clear is that the precedent matters: if copyright litigation can make open-weight releases effectively geography-gated, it changes the calculus for every video model lab with international ambitions.
