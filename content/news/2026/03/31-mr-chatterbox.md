---
title: "What You Get When You Only Train on Public Domain Text"
date: 2026-03-31T07:00:00+00:00
draft: false
slug: mr-chatterbox-public-domain
tags: [training-data, copyright, open-weights]
categories: [safety-policy]
params:
  author: AI Beat Desk
  summary: >-
    Mr. Chatterbox is a 340M-parameter model trained exclusively on 28,000
    Victorian-era texts from the British Library — definitively public domain,
    zero copyright exposure. Simon Willison's writeup documents both what it
    proves and what it falls short of: the corpus is large enough to train
    something coherent, but not large enough to be useful by Chinchilla norms.
---

The training data provenance problem in LLM development hasn't gone away; it's mostly been deferred. Lawsuits grind on. Fair use doctrine is being litigated across multiple jurisdictions. Most labs have responded by training on everything they can reach and hoping the legal environment resolves in their favor.

[Mr. Chatterbox](https://simonwillison.net/2026/Mar/30/mr-chatterbox/) takes the opposite approach: train only on text that is definitively, unambiguously in the public domain. The project pulled 28,000+ Victorian-era books from the British Library — texts published between 1837 and 1899, all of which cleared copyright by a wide margin regardless of jurisdiction. The result is a 340M-parameter model built with Andrej Karpathy's nanochat framework, sitting at 2.05GB on disk, trained on 2.93 billion tokens.

It does not produce useful responses. Simon Willison describes the outputs as closer to Markov chains than coherent language understanding, and his diagnosis points directly at the Chinchilla scaling laws: for 340M parameters, the compute-optimal training volume is somewhere around 6.8 billion tokens. At 2.93B tokens, the model is roughly 60% of the way to its compute-optimal point. Willison estimates it needs 4x more training data to feel like a useful conversational partner.

The problem is that 4x more public-domain text is hard to come by at this quality level. The British Library corpus already represents a substantial slice of what's available in clean, digitized, Victorian-era English. To get another 10–12 billion tokens of similar-vintage public domain text, you'd need to aggregate across Project Gutenberg, the Internet Archive's pre-1928 holdings, and other digitization projects — and even then, you'd be reaching the ceiling of what's available, not just from one source but from all of them combined.

That ceiling reveals something specific about what modern LLMs actually require. The capability gap between a model trained on public domain Victorian prose and a model trained on 2024 internet text isn't purely about scale. It's also about distribution. Conversational patterns, instruction-following norms, technical vocabulary, code, and current events are not well-represented in texts written before the telephone existed. A model trained exclusively on Victorian novels will not learn what a function signature is, what "summarize this" means, or how to answer a question in the imperative register that instruction-tuned models internalize.

This is a useful constraint to examine directly rather than handwave. The copyright-clean floor for LLM training exists but is much lower than the copyright-ambiguous ceiling that frontier labs train against. The difference isn't trivial: it's roughly the difference between a model that can generate plausible Victorian-flavored text and one that can help you debug code.

There's a narrower version of this project that could be genuinely useful: a domain-specific model trained on public domain scientific texts, historical primary sources, or classical literature — where the distribution mismatch with the deployment domain is smaller. A model trained to assist with research on 19th-century British texts, trained on those very texts, would face a much less severe gap between training distribution and use case.

At 340M parameters and 2.93B tokens, Mr. Chatterbox is not that model. But it is an honest experiment with honest documentation of its limits, which is rarer than it sounds.
