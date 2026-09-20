---
title: "Before Jev, There Was Laya"
date: 2026-09-20T06:12:03+00:00
draft: false
slug: laya-before-jev
categories: [inference]
tags: [inference, architecture, open-source, rl-training, structured-output]
params:
  author: AI Beat Desk
  summary: >-
    Four days after TypeSafe AI launched Jev as a breakthrough "System One"
    decision model, a developer surfaced an open-source project called Laya
    built on the same principle a year earlier — faster, more accurate, and
    fully public. The 1,170-point Hacker News thread that followed is less
    about credit disputes and more about what happens when a frontier lab
    takes an existing idea, applies prestige and marketing, and erases the
    prior art.
---

The September 19 post on this site [ended on an open question](/news/2026/09/jev-agent-decision-layer/): whether the next non-autoregressive decision model would come from TypeSafe, from a lab that publishes its training method, or from "someone who figured out that calibration is achievable with a small labeled dataset and the right RL objective." Twenty-four hours later, that someone showed up on Hacker News with 1,170 points.

[Nandakishor M published a post](https://dev.to/nandakishor_m_6cc0adfde9f/i-built-non-autoregressive-decision-models-a-year-ago-then-a-frontier-lab-called-it-a-18me) explaining that he had built exactly this a year ago, called it [Laya](https://github.com/NandhaKishorM/laya), open-sourced it under Apache 2.0, and published all of it — weights, training code, benchmarks — before TypeSafe AI existed as a company. His numbers outperform Jev's published figures: 33 ms inference versus Jev's 150 ms, 76.6% accuracy on the typed-decisions benchmark versus Jev's 72.7%. The project currently has 1.7k stars on GitHub, which in hindsight might have accumulated quietly while everyone was talking about the commercial announcement.

The architecture is specific enough to be useful to read. Laya uses ModernBERT-large (395M parameters) as its backbone — a bidirectional encoder, not an autoregressive decoder — with a purpose-built decision head that resolves `[MASK]` positions against a fixed option vocabulary. The three primitives are `choice` (pick one from a list), `score` (an ordinal), and `noul` (boolean). There's also a multilingual variant using mmBERT-base (322M parameters, 100+ languages) and a fine-tuned `laya-typed-decisions` checkpoint. All three run in a single forward pass; there's no autoregressive decode loop at all.

Training uses what Nandakishor calls RLCD — the same name TypeSafe uses — but he describes the mechanism in detail that TypeSafe hasn't published: proper scoring rules (logarithmic, spherical, and ranked probability scores combined) as the reward signal, with GRPO-style policy gradients on top. Calibration falls out naturally because the reward is specifically constructed so that the model's stated probability is only maximized when it matches the empirical accuracy. A model that says 80% and is right 65% of the time doesn't get credit for confidence it didn't earn. The threshold for autonomous execution — versus routing to a human — is tuned by fitting a cost-sensitive decision head that learns when the confidence score is reliable enough to act on.

The resulting Expected Calibration Error (ECE) is 0.081 after temperature fitting. That's a concrete number TypeSafe hasn't published for Jev at all.

What makes the thread interesting isn't really the priority dispute, though the grievance is legitimate: TypeSafe launched without citing related work, without publishing a paper, without releasing weights, and described the approach as a breakthrough when someone had already done it and given it away. The more revealing part of the discussion is what happened after. Several commenters tried Laya on the free Kaggle fine-tuning notebook linked in the repo and got it working in under an hour. A hardware engineer ran the multilingual variant on a laptop CPU and measured 12 ms cold inference. The entire technical basis for the commercial offering was sitting on Hugging Face under a permissive license. What TypeSafe added was a production API, a polished SDK, the Diogo Almeida founding story (ChatGPT co-inventor), and a manifesto about "System One models" that positioned it as a new category rather than an engineering optimization someone had already built.

None of that is unique to this situation. The gap between research contribution and commercial credit in AI is wide and getting wider. The labs with capital and name recognition get to frame a field as "new" even when the ideas predate them, because discovery credit in machine learning has historically followed funding rounds and conference spotlight slots rather than public repositories. A solo developer publishing a working implementation with benchmarks to a quiet GitHub repo will lose the credit race to a company with a press cycle and a co-inventor of ChatGPT in the founding deck. This is not a new complaint.

What's slightly different here is that Laya is unambiguously better on the metrics that matter for production use: faster inference, better calibration, published ECE, open training code. The argument that TypeSafe's moat is calibration — which we made in these pages — holds less water now that the calibration method has been public for a year and yields lower ECE than Jev's benchmarks imply. The remaining question is whether "production API plus SDK plus enterprise support plus the assurance of a funded company behind it" is worth the cost differential to the teams deploying it. For many of them, it probably is. Infrastructure reliability and warranty of support matter at scale. But that's a product and sales argument, not a technical one.

The open-source clones of Jev that appeared last week were reconstructing something that already existed. Turns out they could have just forked Laya.
