---
title: "The Ghost at the Top of the Rankings"
date: 2026-05-29T06:10:18+00:00
draft: false
slug: hy3-openrouter-mystery
categories: [models]
tags: [tencent, open-source, moe, openrouter, chinese-models, api]
params:
  author: AI Beat Desk
  summary: >-
    Tencent's Hy3 preview — a 295B MoE model with 21B active parameters,
    open-sourced under a community license — has quietly risen to the top of
    OpenRouter's usage rankings, outpacing Claude by over 50%. Almost nobody in
    Western ML circles has written about it. Max Woolf's investigation reveals a
    usage pattern that makes the mystery deeper: 98% input tokens, available
    only through SiliconFlow, and less than 1% of traffic from known apps —
    suggesting a single large unnamed pipeline is driving the entire ranking.
---

Earlier this week, Max Woolf [published an investigation](https://minimaxir.com/2026/05/openrouter-hy3/) into something unusual on OpenRouter's usage charts: a model called Hy3 had climbed to the top of the rankings and stayed there, consuming more traffic than Claude by more than 50%. Woolf had never heard of it. Neither, apparently, had most of the people who read his post.

Hy3 preview is Tencent's latest release from their Hunyuan team — a 295B-parameter Mixture-of-Experts model with 21B active parameters, a 256K context window, and weights available on Hugging Face under the Tencent Hy Community License. [Benchmark numbers](https://www.tencent.com/en-us/articles/2202320.html) are not bad: 74.4% on SWE-bench Verified (up from 53% on Hy2, a substantial jump), competitive with the top tier of open-weight models. It's not an obscure research artifact. It just hasn't generated much Western attention.

The OpenRouter data is what makes the story interesting. A few observations from Woolf's analysis stand out:

The usage pattern skews heavily toward input: roughly 98% input tokens, 2% output. That's a strong signal that whatever is consuming Hy3 is doing batch processing or document analysis, not conversational use. A chatbot produces something closer to a 3:1 or 4:1 input-to-output ratio. Workloads that are 50:1 input-to-output are reading a lot and writing very little — classification, summarization at scale, embedding generation's less sophisticated cousin.

Hy3 is available on OpenRouter only through SiliconFlow, a Chinese inference provider. Less than 1% of the traffic comes from OpenRouter's top tracked apps. That combination — heavy usage, narrow provider availability, almost no fingerprint from known applications — strongly implies that a single unidentified application is responsible for the bulk of the traffic. Someone built something on Hy3 and pointed it at SiliconFlow in bulk.

The economics make this easier to understand. Hy3 on OpenRouter through SiliconFlow prices at roughly $0.066 per million input tokens. For comparison, DeepSeek V4 Flash charges $0.018/M through DeepSeek directly (though the OpenRouter pass-through costs more). At the volumes implied by the OpenRouter rankings, the pricing matters — a few tenths of a cent per million tokens multiplied by billions of tokens is real money. Hy3 is not the cheapest option available, which makes the pricing explanation insufficient on its own; whatever is driving this likely has a specific reason to prefer Hy3 over alternatives.

What that reason is, nobody outside the application knows. Woolf can't identify the app; OpenRouter's public dashboard doesn't expose it. The model tops the charts and the explanation remains invisible.

That opacity is itself worth noting. The narrative about which AI models "matter" is constructed almost entirely from Western developer discourse: benchmark posts, Twitter threads, blog writeups, HN discussions. A model that's generating enormous API traffic through a non-Western inference provider, inside a use case that doesn't produce much visible output, can be genuinely popular in production without appearing in any of those channels. The discourse map and the usage map are different maps.

Hy3 is not a niche or experimental model — it's a serious, well-resourced release from one of the largest technology companies in the world, with competitive benchmark scores and open weights. The fact that it reached the top of OpenRouter's rankings before the English-language ML community had published a substantive post about it says something about the coverage gaps that still exist.

The model is available on [Hugging Face](https://huggingface.co/tencent/Hy3-preview) for anyone who wants to evaluate it directly.
