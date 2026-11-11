---
title: "The Food Delivery Company's Trillion-Parameter Model"
date: 2026-11-11T06:30:00+0000
draft: false
slug: longcat-25-meituan-moe
categories: [models]
tags: [open-source, moe, meituan, inference, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Meituan — better known as China's dominant food delivery platform — posted
    weights for LongCat-2.5 on Hugging Face on November 9 under an MIT
    license. The model uses a mixture-of-experts architecture with roughly
    1.6 trillion total parameters and 48 billion active per token, and its
    self-reported benchmarks put it within one point of frontier closed models
    on agentic coding tasks. The MIT license makes it one of the most
    permissively licensed models at this scale.
---

Meituan is China's dominant food delivery and local services platform — the kind of company where the AI team is a division of a business most people know for restaurant orders and bike rentals. On November 9, that team posted [LongCat-2.5](https://aiweekly.co/alerts/meituan-open-sources-longcat-25-16t-moe-under-mit-license) to Hugging Face under an MIT license: a mixture-of-experts model with roughly 1.6 trillion total parameters and about 48 billion active per token, designed primarily for agentic coding and long-context tasks.

The MIT license is worth pausing on. Most large open-weight releases come with custom licenses that restrict commercial use, derivative models, or redistribution above certain user thresholds. MIT imposes none of that. At this scale — the checkpoint runs about 1.8TB — full MIT is relatively rare, and it matters for companies that want to fine-tune and deploy without legal overhead.

The benchmarks are self-reported and unvalidated, the standard caveat for any new release. The numbers Meituan published are aggressive: 88.4 on Terminal Bench v2.1 (the previous LongCat scored 70.8 on the same benchmark), 64.6 on DeepSWE v1.1 (up from 9.7), and 58.3 on SWE Atlas codebase QA (up from 33.1). Those are large jumps between versions, which could mean genuine capability gains, a shift in training data toward benchmark-adjacent tasks, or some mix of both. The Terminal Bench number puts it within one point of GPT-5.6 Sol and Claude Opus 5, though it trails both on the more recent Terminal Bench v4.0 variant (38.5 vs. 53.9 for Claude Opus 5) and falls behind DeepSeek-V4.1-Flash on DeepSWE (64.6 vs. 74.2).

The architecture uses four-bit expert weights, which makes the active-parameter cost more practical than the trillion-parameter headline suggests. 48 billion active parameters per token puts inference cost in roughly the same range as other large-but-tractable models when you're running at scale on appropriate hardware — which here means hardware that can hold the routing structure and enough experts in memory to avoid constant swapping.

One thing worth watching: the parameter count discrepancy. Meituan announced 1.6 trillion, the Hugging Face checkpoint metadata says 1.8 trillion. Neither side has clarified this yet, and it's the kind of detail that matters for anyone planning infrastructure around the model. Wait for the technical report before treating either number as definitive.

There's also the geopolitical overhang that follows Chinese AI releases right now. U.S. government procurement scrutiny of models from Chinese companies has been tightening, and even MIT-licensed models can face restrictions in certain federal contexts. That doesn't affect most developers, but it's worth knowing if your deployment environment includes government contracts or sensitive infrastructure.

What's interesting about LongCat-2.5 isn't that a food delivery company built a large language model — large tech conglomerates in China have been building AI research divisions for years, and Meituan's team has been publishing since at least 2024. What's interesting is that a model at this scale, with these benchmarks, is now available under a license that lets anyone do anything with it. If the numbers hold up under independent evaluation, that changes the calculus for teams that want frontier-tier capability without frontier-tier licensing restrictions.

The model is in preview status, the technical report hasn't shipped yet, and the benchmark gaps between self-report and independent evaluation can be significant. But the weights are there, the license is permissive, and the first independent evaluations should start appearing over the next week or two.
