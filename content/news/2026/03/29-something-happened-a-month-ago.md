---
title: "Something Happened a Month Ago"
date: 2026-03-29T12:00:00+00:00
draft: false
slug: something-happened-a-month-ago
tags: [open-source, linux, security, code-review]
categories: [safety-policy]
params:
  author: AI Beat Desk
  summary: >-
    Greg Kroah-Hartman at KubeCon EU described an overnight quality shift in
    AI-generated Linux kernel patches — from obvious garbage to ~two-thirds
    correct — that nobody can explain. Simultaneously, Sashiko, an agentic
    patch reviewer from Google's kernel team now hosted at the Linux Foundation,
    is catching 53% of bugs that passed prior human review. AI is entering the
    kernel review pipeline from both directions at once.
---

"Something happened a month ago, and the world switched."

That's Greg Kroah-Hartman, Linux kernel maintainer, at KubeCon Europe last week. He was describing AI-generated security reports. The timeline he sketched: for most of 2025 and into early 2026, the kernel team received what he called "AI slop" — reports that were obviously wrong, full of invented CVEs, hallucinated function names, confident nonsense. Then, abruptly, the quality changed. Real bugs. Actionable patches. The [Register's coverage](https://www.theregister.com/2026/03/26/greg_kroahhartman_ai_kernel/) doesn't pin down a cause, and Kroah-Hartman offered none: "We don't know. Nobody seems to know why. Either a lot more tools got a lot better, or people started going, 'Hey, let's start looking at this.'"

He described running his own experiment with what he called "a really stupid prompt," generating 60 candidate patches. About one-third were wrong. But even those "still pointed out a relatively real problem." Two-thirds were correct. For comparison, that's roughly the pass rate you'd expect from a junior engineer working on an unfamiliar subsystem — not expert work, but not noise either.

The cURL project provides the counterpoint. Earlier this year Daniel Stenberg [stopped paying bug bounties](https://daniel.haxx.se/blog/2026/01/26/the-end-of-the-curl-bug-bounty/) because fewer than 5% of AI-generated submissions were valid. The difference between cURL's experience and Kroah-Hartman's isn't just time — it's also incentive structure. Bug bounties create a direct financial motivation to submit volume regardless of quality. The Linux kernel security list has no bounty; whoever is submitting AI-generated reports there has some filter applied beyond "try everything."

What's interesting about the timing is that it coincides with a second development that's been running in parallel. [Sashiko](https://github.com/sashiko-dev/sashiko) is an agentic patch reviewer built by Roman Gushchin, a Google kernel engineer, and now hosted at the Linux Foundation under Apache 2.0. It takes patches from the kernel mailing list, runs them through a nine-stage LLM pipeline — from analyzing commit goals through security auditing — and returns structured review feedback. In testing against the last 1,000 upstream commits that carried "Fixes:" tags (meaning bugs that were later acknowledged and corrected), Sashiko detected 53.6% of them using Gemini 3.1 Pro. The notable part: all 1,000 of those bugs had already passed human code review. They were in the tree before anyone caught them. Sashiko's false positive rate, measured over a limited manual review, was under 20%.

Fifty-three percent isn't perfect coverage. But "catches more than half of bugs that already fooled every human reviewer before merge" is a genuinely useful operating point. Human review of a Linux kernel patch involves understanding subsystem conventions, implicit invariants, interactions with other recent patches, and kernel-specific safety rules that aren't documented anywhere. That Sashiko can match human reviewers on more than half the bugs that slipped through — using an LLM that has no live kernel state — suggests the detection surface is broader than most people would have expected.

The two threads — better incoming reports, proactive outgoing review — are entering the kernel process at the same time and from opposite directions. External contributors, or at least some subset of them, are generating patches worth reviewing. Sashiko is reviewing patches before they land. Neither is a replacement for the human maintainer judgment that decides what gets merged. But together they're changing the shape of what lands on Kroah-Hartman's desk.

The unexplained quality shift is the most interesting part. AI model capability doesn't step-change on its own between November and February; inference APIs don't get dramatically better without announcement. What does change is who uses them and how. The most plausible explanation isn't that models improved — it's that a critical mass of people who actually understand kernel internals started using these tools seriously, and the submissions from that population now dominate the signal. The slop didn't disappear; it just got diluted by enough real work that the ratio flipped.

Whether that holds is a different question. The economic incentive to submit AI-generated vulnerability reports at scale is real, and the defenses against it — the bug bounty shutdown, moderation queues, endorsement requirements — are reactive by nature. For now, the kernel team got lucky: the use case attracted people who knew what they were doing. That's not guaranteed to last.
