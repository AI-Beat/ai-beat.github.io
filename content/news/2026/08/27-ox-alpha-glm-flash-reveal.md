---
title: "Ox Alpha Was Zhipu All Along"
date: 2026-08-27T06:13:51+00:00
draft: false
slug: ox-alpha-glm-flash-reveal
categories: [models]
tags: [glm, zhipu, moe, open-weights, multimodal]
params:
  author: AI Beat Desk
  summary: >-
    Z.ai reveals that Ox Alpha — the anonymous model circulating on OpenRouter and OpenCode for a week — is GLM-5.3-Flash: a 320B/18B mixture-of-experts with linear attention, native multimodal input, and an MIT license. It ran the whole time on domestically produced Chinese chips.
---

On August 20, a model called "Ox Alpha" appeared on OpenRouter listed under a stealth provider with no attribution. It was free, had a 1M-token context window, accepted text, images, and video, and immediately attracted agentic traffic. For a week it ran anonymously, building a track record without any lab claiming credit for it.

Yesterday, Z.ai ended the ambiguity: [Ox Alpha is GLM-5.3-Flash](https://z.ai/blog/glm-5.3-flash). It's the company's first natively multimodal entry in the GLM-5 line, and unlike the base GLM-5.3 — which is still under a more restricted license — this variant ships under MIT with weights on Hugging Face.

The specs are interesting on their own. It's a 320B-total / 18B-active mixture-of-experts, trained on 30 trillion tokens, with a 1,048,576-token context and up to 131K tokens of output. The architecture combines two memory optimizations: sparse attention (attending only to relevant tokens rather than the full sequence) and linear attention (replacing softmax with an approximation that brings memory from \(O(n^2)\) to \(O(n)\)). Z.ai also mentions mHC — a technique applied during training to prevent gradient distortion in sparse routing. Together, these let the model handle the million-token window at a fraction of the memory cost a standard transformer would need.

On benchmarks: GLM-5.3-Flash lands first on GDPval-AA v2 (knowledge work evaluation), second on AutomationBench (cloud application task completion), and 84.3 on Terminal Bench 2.1, which puts it within striking distance of Claude Opus 4.8's 85.0. DeepSWE v1.1 climbs from 46.2 to 63.4 versus the non-Flash GLM-5.3, and AutomationBench nearly doubles from 26.2 to 48.8. As always, these are vendor-reported numbers against vendor-selected baselines — treat them as Z.ai's best case.

What makes the anonymous week worth noting isn't just clever marketing. Ox Alpha accumulated genuine production traffic on OpenRouter's infrastructure before any lab attached its name to the output. That's a form of live evaluation — unbranded, running against real user queries — that most model releases skip entirely in favor of benchmark tables written at release time. Whether the results are representative is a separate question, but the approach is unusual enough to be worth tracking as a strategy.

There's also the chip story. Z.ai says GLM-5.3-Flash ran its silent week entirely on domestically produced Chinese AI hardware — not H100s or B200s. The model sat in production under real agentic load on a supply chain that's fully insulated from US export controls. The inference performance was apparently fine enough that no one noticed the infrastructure difference.

Pricing lands at $0.15 per million input tokens and $0.50 per million output tokens at full rate, with a 50% promotional discount through September 9 — so $0.075 input, $0.25 output currently. That puts it well below the price of the non-Flash GLM-5.3 and in a competitive range for a 1M-context multimodal model.

The weights are [available on Hugging Face](https://huggingface.co/THUDM) under MIT, which means fine-tuning and local deployment are on the table for anyone with the hardware to run a model this size.
