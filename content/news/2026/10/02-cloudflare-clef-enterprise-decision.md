---
title: "Cloudflare Enters the Decision Model Market"
date: 2026-10-02T07:00:00+02:00
draft: false
slug: cloudflare-clef-enterprise-decision
categories: [inference]
tags: [inference, classification, agents, open-source, rl]
params:
  author: AI Beat Desk
  summary: >-
    Cloudflare released Clef and Clef-flash, open-weight decision models at 27B
    and 9B for agent routing and classification, alongside a managed RL
    fine-tuning service. The release validates the decision model category at
    enterprise scale, while raising the usual question about whether managed
    cloud infrastructure beats the increasingly capable self-hosted alternatives.
---

Over the past few weeks, the community has been quietly assembling a decision model stack. [Jev](https://ai-beat.github.io/news/2026/09/19/jev-agent-decision-layer/) defined the classification interface. [Ollaya](https://ai-beat.github.io/news/2026/09/26/ollaya-local-decision-models/) packaged Jev-compatible models for local deployment. [Jeff](https://ai-beat.github.io/news/2026/09/29/jeff-decision-model-calibration/) showed you could train them at home on a single GPU, hitting Jev-level accuracy at 2B parameters in a few hours of fine-tuning.

Now Cloudflare is entering the space with [Clef](https://blog.cloudflare.com/clef-decision-models/), and the release is interesting for what it says about where the category is heading.

Decision models are not generative. They don't produce free-form text — they return typed outputs with probability scores. A decision model answers questions like "which of these four agents handles this task?", "does this image require moderation?", or "should this user message trigger a tool call?" Eliminating token-by-token generation makes them deterministic and cheap: the classification happens in one forward pass with no sampling.

Cloudflare released two models: Clef (27B, built on Qwen 27B) and Clef-flash (9B, 38.8ms median latency). Both are on Hugging Face under Apache 2.0 and hosted directly on Workers AI. Both accept up to four images alongside text input — the vision capability is a differentiator from most community models in this space, which have focused exclusively on text. The 64k context window covers most classification scenarios without truncation.

On Cloudflare's internal benchmarks, Clef outperforms Jev on most tasks. That comparison is somewhat expected given the 27B vs. much-smaller size gap, and independent evaluation will be more informative. The Clef-flash latency numbers are more immediately meaningful: 38.8ms for a routing decision is fast enough to be invisible in most orchestration loops.

The more interesting part of the announcement is the RL fine-tuning service. Decision model performance is heavily domain-specific. A model trained on synthetic routing scenarios may not transfer well to your particular classification schema. The community approach is to fine-tune on your own data on your own hardware from an open base. Cloudflare's managed offering runs fine-tuning on its Containers infrastructure, collects labeled decisions through AI Gateway, and redeploys the fine-tuned model directly onto Workers AI — with no exfiltration of labeled data. Currently this requires working with Cloudflare's forward-deployed engineering team; a self-serve path is planned.

The self-hosted vs. managed question is real here. A Jeff-sized 2B decision model runs on consumer hardware, costs nothing per call, is private by definition, and is cheap to fine-tune. The accuracy ceiling is lower, but for most routing tasks that ceiling isn't the constraint. Cloudflare's 27B Clef offers meaningfully better accuracy and adds vision — at the cost of cloud dependency and whatever pricing eventually looks like.

What Cloudflare's entry signals is that decision models are becoming infrastructure-level. When a major CDN and edge compute provider releases open-weight classification models and a fine-tuning service on the same day, the architectural pattern has probably crossed from "niche research" to "component engineers expect to have available." The community was already there; the enterprise layer is catching up.
