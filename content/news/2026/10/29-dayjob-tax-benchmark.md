---
title: "What Issue Spotting Costs"
date: 2026-10-29T06:16:00+00:00
draft: false
slug: dayjob-tax-benchmark
categories: [evaluation]
tags: [evaluation, agents, benchmarks, professional]
params:
  author: AI Beat Desk
  summary: >-
    Surge HQ's DAYJOB: Tax benchmark tests AI agents on real professional tax
    work — not answering given questions, but identifying the issues independently
    from messy client files. Claude Sonnet 5.5 led with 10.8% under a 95%-criteria
    threshold. The benchmark is interesting because it measures something most evals
    skip: the ability to notice what the problem is before solving it.
---

Most AI benchmarks are question-answering at heart. You hand the model a problem; the model solves it. The benchmark tells you how often the answer is right. What it can't tell you is whether the model would have noticed there was a problem in the first place.

[Surge HQ's DAYJOB: Tax](https://surgehq.ai/blog/dayjob-tax), published today, is built around that gap. The benchmark covers 50 professional tax environments — 1,789 files across eight practice areas including individual, small-business, partnership, and international tax. Agents receive a folder of client documents and must do what a tax professional does: find the issues, including the ones the client never mentioned, and deliver finished work at professional quality. Passing requires meeting at least 95% of a task's criteria.

The results: Claude Sonnet 5.5 led at 10.8%, followed by Claude Fable 5.1 at 9.6% and Opus 5.5 at 9.2%. Under a 95% threshold, that's a tight ceiling. The benchmark's summary is that agents "haven't yet passed the audit."

The specific failure patterns matter more than the top-line numbers. In one case, agents evaluated two sale dates on a condo correctly — they got the computation right. But they missed that the owner's eligibility for a capital gains exclusion would expire in August 2027, a future-contingent issue that required connecting the transaction dates with a regulatory threshold the client hadn't raised. Opus caught it in all five of its runs; Astra never did.

In another case, agents had to advise a separating couple on retirement account treatment. Multiple models consistently assumed the couple was unmarried, even though the case documents included records suggesting a possibly valid marriage from another state. Opus checked marital status in all five runs; another model checked in four of five. But the point is that checking at all — noticing that the assumption matters — is the professional skill being tested, and it requires something different from calculation.

This kind of "issue spotting" is what distinguishes professional practice from problem-solving. A tax professional working a real client engagement doesn't receive a well-formed question. They receive a collection of documents, some of them contradictory, some of them incomplete, and they need to identify what matters before they can advise on it. DAYJOB's benchmark inputs are designed to reflect this: handwritten receipts, dispatch logs showing work in five states, conflicting records — not because the task designers wanted to be difficult, but because that's what the actual work looks like.

The 10% ceiling isn't evidence that the benchmark is broken. It's measuring a capability that most benchmarks actively avoid measuring, because it requires the evaluator to know what issues exist in a scenario, independent of whether the agent found them. That's hard to construct at scale, which is probably why there are so few benchmarks that do it.

The framing matters for how you interpret current AI capabilities. Models that score well on question-answering benchmarks can look ready for professional work. DAYJOB's results suggest that the jump from "answering tax questions" to "doing tax work" — where you have to first know which questions to ask — is larger than the headline benchmark numbers imply. The question being evaluated shifts from "can the model solve this?" to "does the model notice this needs to be solved?" Those are different things, and the second one is harder.

At 10%, top models are not useless — they're catching some consequential issues some of the time. But a professional who misses the August 2027 exclusion cliff isn't doing their job, regardless of what else they got right. That's the bar the benchmark sets, and it's a reasonable one.
