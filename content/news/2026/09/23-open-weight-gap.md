---
title: "Open Weights, Unequal Outcomes"
date: 2026-09-23T06:20:52+00:00
draft: false
slug: open-weight-gap
categories: [models]
tags: [models, open-source, china, benchmarks, llm]
params:
  author: AI Beat Desk
  summary: >-
    Nathan Lambert's analysis, structured as Congressional testimony, puts
    hard numbers on what practitioners already sensed: Chinese open-weight
    models have pulled ahead of American ones on capability, downloads, and
    academic adoption. Chinese open models score 42–45 on the Artificial
    Analysis Intelligence Index; American open models score 23–26. The
    distillation argument explains only 1–2 months of the gap.
---

Nathan Lambert [published an essay on September 21](https://www.interconnects.ai/p/the-current-balance-of-power-in-open) framed as Congressional testimony on U.S.–China AI competition in open-weight models. The framing is unusual; the underlying argument is not. By nearly every metric Lambert tracks, Chinese open-weight models have surpassed American ones and the gap is large.

The benchmark comparison is the sharpest signal. The top Chinese open models — Z.ai's [GLM-5.3](https://artificialanalysis.ai/) at 45, GLM-5.3-Flash at 42, and Moonshot AI's [Kimi K3](https://kimi.moonshot.cn/) at 44 — score substantially higher on the Artificial Analysis Intelligence Index than the leading American open models, which cluster around 23–26. That is a roughly 20-point spread. Kimi K3 is a 2.8T parameter sparse MoE with 104B active parameters and 1M context, weights published in July; GLM-5.3 comes from Z.ai (formerly Zhipu AI). These are not boutique releases.

Download counts tell the same story from a different angle. Chinese models have been downloaded approximately 3.2 billion times on Hugging Face versus 1.6 billion for American models. On OpenRouter, Chinese models account for roughly 80% of open-model inference traffic — a distribution that reflects where practitioners are placing real workloads, not just where they are experimenting.

Academic adoption mirrors the deployment picture. Lambert estimates Chinese models appear in about 40% of recent AI research papers versus 30% for American ones. Qwen specifically is cited in 30% of papers; Llama, the dominant American open-weight family, is cited in 21%. This matters because academic papers set norms for future work: the models that researchers build on tend to be the ones that accumulate capability improvements and community ecosystem.

Lambert addresses the distillation objection directly: some fraction of Chinese open-model capability comes from training on outputs of American closed models (OpenAI, Anthropic), so the comparison is partly circular. His estimate is that distillation accounts for only 1–2 months of the Chinese competitive advantage, leaving the bulk of the gap unexplained by that mechanism. American frontier models (GPT-6 Astra, Claude Fable) remain ahead of the best Chinese open models, but that lead has been declining; Lambert puts it at 2–5 months. American open-weight models trail their own closed frontier by 6–9 months.

This is worth reading carefully: American open-weight models are behind both Chinese open models and American closed ones. The competitive pressure on American open-weight efforts — concentrated almost entirely in Meta's Llama line plus a handful of smaller players — comes from two directions simultaneously.

The supply-side explanation is visible in the landscape: [Qwen/Alibaba](https://qwen.ai/), DeepSeek, Moonshot AI, Z.ai, [Xiaomi's MiMo](https://mimo.mi.com/) — multiple Chinese labs are each independently releasing frontier-competitive open-weight models. The Chinese open ecosystem has more contributors at the capability frontier, each following different architectural approaches, all releasing weights. That structural difference is harder to close with a model release than a benchmark gap is.

There is a counterpoint worth noting. Chinese labs control the openness decision: [Qwen-Image-2.1's shift to a research-only license](https://qwen.ai/blog?id=qwen-image-2.1) two days ago is a reminder that weights published today can be re-licensed or restricted tomorrow. Pirate Face and similar preservation efforts exist precisely because the "open" part of open-weight is contingent. The download numbers and inference market share represent the current moment, not a permanent commitment.

For engineers selecting models: if you are deploying open-weight models and optimizing for capability per dollar today, the top Chinese models are the competitive options. Lambert's analysis frames this as a policy concern, and at the national scale it may be. At the workload level, it is just a statement about where the best weights are.
