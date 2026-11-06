---
title: "One Training Run, Native 4-Bit"
date: 2026-11-06T07:00:00+01:00
draft: false
slug: inkling-2-small
categories: [models]
tags: [open-weights, MoE, RL, training, efficiency]
params:
  author: AI Beat Desk
  summary: >-
    Thinking Machines Lab released Inkling 2 Small, a 594B-parameter MoE model with 17B active parameters, pretrained natively in NVFP4 and post-trained with a single joint reinforcement learning run of 1,075 steps across 17 million rollouts. Weights are closed pending external safety review; Apache 2.0 is planned.
---

[Thinking Machines Lab](https://thinkingmachines.ai), a Philippine AI research company, released [Inkling 2 Small](https://thinkingmachines.ai/news/inkling-2-small/) on November 5. The model is a 594B-parameter mixture-of-experts with 17B active parameters per token—a sparsity ratio of roughly 35×, more aggressive than most published MoEs. Pretraining ran in multiple phases, with the first covering 8.4 trillion tokens and subsequent phases adding several trillion more.

Two training choices stand out.

The first is NVFP4 precision from the start. Most models are trained in BF16 or FP8 and quantized afterward for deployment. Inkling 2 Small was pretrained natively in NVFP4 on NVIDIA GB300 NVL72 racks, meaning the quantization was not applied as a compression step but was part of the original compute budget. The company says this produced a model that is easier to fine-tune than its predecessors at the same task—presumably because the weights never learned to rely on higher-precision representations that disappear at quantization. Whether the NVFP4-native approach generalizes to other architectures remains an open question, but it is a data point worth watching.

The second is the reinforcement learning setup. Post-training used a single joint RL run of 1,075 steps spanning 17 million rollouts, applied to agentic coding, vision, and reasoning simultaneously rather than in separate domain-specific phases. Running a single continuous RL training rather than chaining together separate specialists is harder to orchestrate but avoids the reward interference and forgetting that can appear when objectives are applied sequentially. The company reports that this contributed to better generalization across domains.

The model accepts image inputs and supports variable reasoning effort. Benchmarks are reported for agentic coding tasks, finance-domain structured reasoning, vision (OCR, grounding, visual QA), and knowledge work. Thinking Machines says Inkling 2 Small achieves the best published scores among open-weight models on several finance benchmarks. The company is based in Southeast Asia and has historically focused on regional enterprise applications, so the finance emphasis reflects their deployment context as much as a general research bet.

The wine identification demo—correctly identifying three wines from a single photo in 19 of 20 trials—reads more like a marketing choice than a research contribution, but it gestures at something real: the model can connect visual attributes to named real-world objects in a way that many vision-language models still handle inconsistently.

Current access is through the company's Tinker platform with closed weights. The weights will be released under Apache 2.0 after external safety testing and red-teaming, the company says, with no specific date given. The gap between "we have weights" and "weights are publicly available" has been variable across labs, so the timeline is worth following.

The broader point is that Inkling 2 Small continues a trend that has been running for a year: labs outside the US and EU tier are building models that are competitive at specialized tasks, often by making architectural or training choices that more resourced labs have not prioritized. NVFP4-native training and a single joint RL run are both choices that imply deep integration with the hardware and training stack, and both reflect a team that has thought carefully about what matters for their deployment context rather than simply scaling a reference architecture.
