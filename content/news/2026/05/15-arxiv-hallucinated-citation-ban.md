---
title: "arXiv's Citation Crackdown"
date: 2026-05-15T06:15:00+00:00
draft: false
slug: arxiv-hallucinated-citation-ban
categories: [research]
tags: [hallucination, arxiv, academic-publishing, integrity, policy]
params:
  author: AI Beat Desk
  summary: >-
    arXiv began enforcing a new policy this week: submit a paper with
    AI-hallucinated citations and you're banned from the platform for a year,
    after which future preprints require peer-review acceptance before posting.
    With fabricated citations rising tenfold since 2023 — now appearing in 1 in
    277 papers — arXiv's response is to repurpose the peer-review gate that most
    researchers treat as optional into a punitive instrument.
---

Sometime this week, arXiv updated its enforcement posture on a problem that has been quietly worsening for three years. The new rule is simple: submit a paper containing AI-hallucinated citations — references to papers that don't exist — and you're banned from the platform for a year. After that year, you lose the preprint-first privilege entirely; future submissions must be accepted at a reputable peer-reviewed venue before you can post them.

The numbers that prompted this are worth sitting with. In 2023, roughly 1 in 2,828 arXiv papers contained a hallucinated reference. By 2025 that had risen to 1 in 458. By early 2026, it was 1 in 277 — a tenfold increase in three years, tracking almost perfectly with the adoption curve of LLM writing tools. At NeurIPS 2025, analysis of accepted papers found at least 100 confirmed hallucinated citations spread across 53 papers. At ICLR 2026, 20% of a sampled subset — roughly 60 out of 300 papers examined — contained at least one AI hallucination of some kind.

arXiv's stated rationale puts the responsibility squarely on authors: "By signing your name as an author of a paper, each author takes full responsibility for all its contents, irrespective of how the contents were generated." The platform frames it as accountability, not prohibition. You can use LLMs. You just have to check what they produce.

The policy has a built-in enforcement problem that critics have already surfaced: arXiv processes thousands of submissions per day and has no automated hallucination-detection layer for citations. Enforcement presumably relies on community flagging, post-publication verification, or complaints from cited authors who notice their work being mis-cited or invented. That's a slow and incomplete signal. The policy may deter the careless more than it catches the reckless, since it requires "incontrovertible evidence that the authors did not check the results of LLM generation" before applying penalties — a bar that's genuinely hard to clear from the outside.

But the structural design of the penalty is cleverer than it first appears. arXiv's value to researchers is precisely that it bypasses the peer-review queue. You can post a preprint today and have it indexed, shared, and cited within hours. Peer review takes months at most venues, sometimes years. Using mandatory peer-review acceptance as a punitive gating mechanism isn't just a punishment — it's a targeted removal of the thing that makes arXiv worth using. The message to repeat offenders is: you've lost the express lane.

There's a subtler point embedded in this, too. The tenfold increase in hallucinated citations correlates directly with LLM writing assistance, but fabricated references were a known problem in academic writing well before ChatGPT. Lazy citation practices — citing papers you haven't read, copying reference lists from other papers, forwarding chains of secondary citations — have always existed. LLMs didn't create the problem; they industrialized it. The [HalluCitation study from January](https://arxiv.org/abs/2601.18724) found 300 such papers in ACL conference proceedings over a five-year window, including pre-LLM years. arXiv's policy is responding to a rate change, not a new category of error.

The community reaction is mixed in predictable ways. Many researchers welcome it as overdue quality control — arXiv is a privilege, not a right. Others worry about junior researchers whose advisors submit work without adequate verification, or about the difficulty of distinguishing honest errors from negligence. Both concerns are reasonable. Neither undermines the core point: if citation integrity is degrading at a measurable, accelerating rate, some institutional response is necessary. Whether arXiv's particular instrument is calibrated correctly will become clear as enforcement cases accumulate.

For now, the practical upshot is this: if you're using any LLM for writing assistance, verify every reference it touches. Not spot-check — verify. A citation that looks plausible, formatted correctly, in the right venue, with a realistic author list, can still be entirely fabricated. The model doesn't know; it's pattern-matching against a distribution of what citations look like, not retrieving from a verified database. The burden of checking has always been on authors. arXiv just made the cost of not checking much higher.
