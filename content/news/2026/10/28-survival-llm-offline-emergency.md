---
title: "What You Know When the Internet's Gone"
date: 2026-10-28T06:13:00+00:00
draft: false
slug: survival-llm-offline-emergency
categories: [tools]
tags: [local-inference, offline, rag, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Survival LLM is a free, open-source offline AI assistant bundled with roughly
    930,000 Wikipedia articles, medical references, FEMA guidance, and U.S. military
    survival manuals. It selects a model tier based on your machine's RAM and runs
    fully locally after the initial download. The interesting design question isn't
    the AI — it's what you bundle with it.
---

Most "local AI" projects are about privacy or cost. [Survival LLM](https://www.brangus.io/survival-llm) is about something different: what happens when the internet is gone and you need to know something important.

The project, which shipped version 0.3.0 on October 26 and iterated to 0.3.4 by October 27, is a macOS and Windows app that bundles an LLM with an offline reference library and runs entirely on your own hardware. After the initial setup (which needs internet), it works without any connection. Nothing you ask leaves the machine.

The knowledge base is the interesting part. It includes roughly 930,000 of Wikipedia's most-read articles, WikiMed, MedlinePlus, UK Medicines A to Z, and WikEM for medical topics, Ready.gov and CDC travelers' health guidance, U.S. military medicine and survival manuals, and wikiHow, iFixit, Appropedia, and Stack Exchange Q&A on home repair, cooking, gardening, and amateur radio. That's a specific, considered selection — not a dump of Common Crawl, but curated sources weighted toward practical knowledge under pressure.

The model selection is tiered by RAM: Apple Silicon Macs with 128 GB or more get NVIDIA Nemotron 3.5 Super; 16–31 GB machines get IBM Granite 4.2 8B. The app detects available memory at launch and downloads accordingly. Each answer cites the articles it used, with the relevant passages highlighted when you open them. You can also drop in your own PDFs, ebooks, and notes, which get indexed locally for the same retrieval.

The retrieval is local and fast — the page claims sub-second search across the full library, which is plausible given plain-text indexes and a document set that, while large, is entirely static.

What Survival LLM gets right is framing the knowledge base as the product, not the model. The LLM is the interface — it helps you ask questions and synthesize answers in natural language — but the actual value is the curated offline corpus. A 2020-era LLM with good retrieval over the right documents would serve most survival queries adequately. The model quality matters for synthesis and follow-up questions, but the bottleneck in a real emergency is usually the knowledge itself, not the reasoning quality.

That's a useful inversion of how most RAG systems are designed. Most retrieval-augmented systems treat the corpus as the variable you tune around a fixed model. Here the corpus is the thing you ship, and the model is tiered to match available hardware. The corpus ships once and doesn't need updating; the model tier adapts to what you have.

The obvious limitation is platform: macOS on Apple Silicon (M1 or newer), or Windows 64-bit. No Linux, no Android, no web fallback. For someone who actually wants this as a resilience tool — a generator, an off-grid homestead, a field kit — the platform constraint might be the practical blocker.

The [project is open source](https://www.brangus.io/survival-llm), which means someone will presumably port the knowledge-base curation and bundling approach to other platforms eventually. The more durable contribution might be the selection criteria: what sources are worth indexing offline for emergency use, and how to keep them compact enough to distribute.
