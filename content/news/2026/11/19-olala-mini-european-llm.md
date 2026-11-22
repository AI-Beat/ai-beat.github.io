---
title: "Olala: OVHcloud's Open LLM and the Question Nobody Asked"
date: 2026-11-19T06:00:00Z
draft: false
slug: olala-mini-european-llm
categories: [models]
tags: [open-source, moe, europe, inference]
params:
  author: AI Beat Desk
  summary: >-
    OVHcloud releases Olala mini, its first open-weight LLM: a 7B MoE with
    hybrid Differential-TPA and Mamba-3 attention, Apache 2.0 license, trained
    on European supercomputers — and conspicuously no benchmark comparisons
    published. The more interesting story is why a French cloud provider is
    building its own model at all.
---

The name alone — Olala — is a bit of a provocation. OVHcloud, France's largest cloud provider, launched its first open-weight language model today at the OVHcloud AI Summit in Paris, with weights landing on Hugging Face under Apache 2.0 by end of day.

Architecturally, Olala mini is a 7-billion-parameter mixture-of-experts model with roughly 1 billion active parameters per token, organized across 256 experts per layer. Its attention layer combines two mechanisms: Differential-TPA, a compressed and filtered variant of classical attention, and Mamba-3, which processes sequences as a stream with a fixed-size state summary rather than a full quadratic attention over the context. This hybrid is designed to let the model run efficiently on modest hardware — OVHcloud claims 160 tokens per second on an RTX A4500 (a 20GB card) and 50 tokens per second without any GPU at all. The architecture came with [Dragon LLM](https://af.net/realtime/ovhcloud-releases-open-weight-7b-olala-mini-llm-under-apache-2-0/), a startup OVHcloud acquired as part of this effort. Training used approximately €7 million in compute time donated across the Jupiter, MeluXina, and Jean Zay European supercomputers.

What OVHcloud did not publish: any benchmark comparison against other models. No MMLU, no HumanEval, no head-to-head. The company says V1.1 — due before year-end, with a 128K context window — will come with proper evaluations. The only performance number on offer is a customer reference: Qwant, the French search engine, reports that Olala mini makes its flash answer feature 20% faster and cuts inference costs to a third of its previous model's. The catch is that Qwant is owned by Synfonium, whose majority shareholder is the Klaba family — who also run OVHcloud — so the data point is not independent.

The absence of benchmarks could be read as humility (don't hype what you can't prove) or as a hedge (numbers might not look flattering against models trained on much more compute). The honest version is probably both.

What makes this release genuinely interesting isn't the model itself — a 7B MoE at this size is not remarkable in 2026 — but [the stated motivation](https://itdaily.com/news/software/ovhcloud-olala-mini-european-language-model/). CEO Octave Klaba said directly: "We cannot keep relying on the generosity of certain labs to share their language models as open source in the coming years." That's a real concern. The last eighteen months have seen a few major open-weight releases pulled behind commercial licenses or delayed indefinitely, and the open-source ecosystem has largely been sustained by a handful of large labs making strategic bets. A cloud provider in Europe deciding it needs its own model family, trained on public supercomputer infrastructure, is a different kind of commitment — it's infrastructure sovereignty rather than research output.

The [AI Act compliance documentation](https://www.brusselstimes.com/2410206/french-company-launches-ai-model-olala-as-rival-to-us-and-chinese-competitors/) is also worth noting: under EU law since August 2025, any general-purpose AI model provider must publish a copyright policy and a summary of training content, even for open releases. OVHcloud is publishing that documentation alongside the weights. It's a small thing but it sets a precedent for what compliant open-source model releases look like in Europe.

Olala mini's V2 roadmap calls for a model over 100 billion parameters, with no release date attached. The more immediate test will be whether V1.1's promised benchmarks hold up and whether the 128K context window makes it useful for more than fast search summaries. Right now, Olala mini is a credible infrastructure play wrapped in a model that hasn't been stress-tested in public yet.
