---
title: "Pages Nobody Was Meant to Read"
date: 2026-09-03T06:09:00Z
draft: false
slug: manufactured-sources-ai-citations
categories: [industry]
tags: [rag, retrieval, perplexity, grounding, web-ecosystem]
params:
  author: AI Beat Desk
  summary: >-
    Trellner Research tested Perplexity's Sonar models across 380 software categories
    and found three newly-registered domains had published 215,128 machine-generated
    "best software" pages — HTML-titled "Facts & Grounding Page" — that Perplexity cites
    extensively. 59.8% of its citations lead to domains outside the global top-100,000.
---

When Perplexity recommends software, where does the evidence come from? [Trellner Research](https://trellner.com/reports/manufactured-sources-behind-ai-recommendations/) spent time mapping that question across 380 buyer-intent software categories, submitting 760 queries (two Sonar model variants, same category set, each asked to return a ranked top-five as JSON) and tracking every URL that came back.

The citation analysis is not flattering. Of the 7,534 citations returned, 59.8% point at domains ranked worse than 100,000th on the [Tranco](https://tranco-list.eu/) top-1 million list — a rough proxy for global traffic. About 23.4% don't appear in the top million at all. The median cited domain has a Tranco rank of 71,611. Not fringe, exactly, but well outside the universe of sources a professional researcher would recognize.

The main story is three domains that appear repeatedly across categories: wifitalents.com, worldmetrics.org, and gitnux.org. Operating under apparent common control, all registered between December 2023 and May 2024, sharing Cloudflare nameservers and identical page templates, they have collectively published 215,128 pages optimized for software category queries. The Wayback Machine confirms none of them existed before the current generative AI wave.

Here is the detail that makes this more than a routine quality complaint: both sites with a homepage have titled it "Facts & Grounding Page" in their HTML `<title>` tag. "Grounding" is the technical term for the retrieval step that RAG-based systems use to find supporting evidence before generating a response. That title was not written for human readers. It's a bid to look authoritative to a vector search pipeline.

The deeper issue isn't that these three sites represent unusual sophistication. They're almost trivially simple. The issue is that they demonstrate the obvious consequence of AI systems becoming dominant information intermediaries: the incentive for content creation shifts from writing clearly for humans to writing in a way that retrieval systems will surface. Search engine optimization spent years corrupting the web's information ecosystem, but it was at least aimed at human click-through. These pages have entirely abandoned the human reader as the audience. They're pure pipeline content.

A handful of citations in the Trellner dataset illustrate failure modes well. Some recommended vendors' homepages have gone offline; others redirect to unrelated businesses. One citation resolved to an Indonesian gambling portal. Perplexity's retrieval pipeline isn't checking link validity or domain longevity; it's matching embeddings and surfacing whatever fits.

The immediate problem — that AI-assisted purchase decisions route through content factories — is solvable in principle with better freshness filtering, stricter domain-quality scoring, and factual cross-checking against known product registries. The harder problem is structural. Once it's established that AI grounding systems select on retrieval-signal quality rather than source quality, the content incentives change for everyone, not just the operators of three fast-registered domains. If the signal you need to produce is "looks authoritative to a dense retrieval model," you optimize for that signal. "Facts & Grounding Page" is just someone saying the quiet part in the page title.

The Trellner report also implies something about the feedback loops that concern people building web-trained models. Training data quality is usually discussed in terms of toxicity, copyright, or factual accuracy. This is a different vector: the web's content is being shaped by what AI systems surface, and AI systems are being trained on that content. That loop is not new — SEO has been running it for decades — but generative AI is running it faster and with lower production cost per page.

Perplexity has not commented on the report specifically. The broader category of problems it illustrates — AI systems whose grounding evidence is itself AI-optimized rather than evidence-grounded — is not going away.
