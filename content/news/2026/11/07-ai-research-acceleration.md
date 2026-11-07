---
title: "The 2× Signal"
date: 2026-11-07T07:00:00+01:00
draft: false
slug: ai-research-acceleration
categories: [safety-policy]
tags: [safety, RSP, alignment, acceleration, METR]
params:
  author: AI Beat Desk
  summary: >-
    METR researcher Ajeya Cotra argues from public evidence that METR's November
    assessment found roughly 2× speedup in Anthropic's capabilities progress,
    mostly AI-attributable, which likely crosses the RSP threshold for "dramatically
    accelerated" progress — and that the industry is nowhere near ready for what
    that implies.
---

Anthropic's amended S-1/A, filed November 2, contains an unusual risk-factor line: the company says it "may imminently meet" the threshold it defined in its own Responsible Scaling Policy for when AI begins dramatically accelerating AI research. Two days later, METR published an assessment of Anthropic's internal capabilities progress, drawing on elevated access to internal data. Yesterday, Ajeya Cotra — a METR researcher known for her work on AI timelines and recursive self-improvement risk — wrote [a detailed public analysis](https://www.planned-obsolescence.org/p/ai-is-dramatically-accelerating-ai) of that METR data, arguing the threshold has probably already been crossed.

This is unusual enough that it's worth unpacking what all three of those documents actually say.

## What the RSP threshold means

Anthropic's RSP defines "dramatically accelerated AI-attributable progress" as occurring when capabilities progress is roughly twice as fast as both the historical baseline and what it would be absent significant AI contribution, and when that speedup is plausibly attributable to AI automating research or engineering work in a way that could keep compounding.

That definition is careful on purpose. Progress accelerates for other reasons — more compute, more engineers, better data. The RSP threshold is specifically about progress that AI itself is driving, in a feedback loop. The concern is not just speed; it's compound growth in a system that's improving itself.

## What METR found

METR's report (published November 5) measured Anthropic's capabilities progress from October 2025 to October 2026 using an internal version of the Epoch Capabilities Index, a composite index of AI task performance. Progress over that twelve-month window ran about twice as fast as in the prior twelve months. METR attributed the bulk of that speedup — roughly 80% — to AI automation of research work, not to other factors. It found no clear bottlenecks suggesting natural diminishing returns soon.

METR's estimate of the AI-attributable multiplier, as of late October, was roughly 1.8×. That falls just below the nominal 2× threshold, but Cotra points out it was trending upward and was likely higher by the time the post went out. Much of the supporting evidence is redacted.

Cotra also does her own independent estimate from first principles. If compute is multiplied roughly 2.5–3× over the year and overall progress doubled, then algorithmic progress must have sped up 2–3×. For algorithmic progress to accelerate 2–3×, researcher labor must have been lifted much more than that — her estimate is roughly 10× effective labor uplift when diminishing returns are factored in. Anthropic staff self-reports of a ~12× uplift from internal models are roughly consistent with that number.

## What compounds

The concerning part of Cotra's argument is not the current 2× figure — it's what a 2× speedup implies if it keeps compounding. Under the RSP definition, the threshold is designed to be an early warning trigger, not a point-of-no-return marker. The problem is that once AI is contributing materially to its own improvement, the feedback starts accelerating.

Cotra's scenario: if the doubling time for capabilities progress starts at six months (as it roughly is now), and AI-driven acceleration steadily grows, the doubling time could fall to three months, then six weeks. That's a rough exponential that, if it held, would imply extreme capabilities within a year. She is explicit that she expects real limits and that the scenario is not certain — but she argues there's no obvious reason to assume the limits are close.

## Why this post exists

What's worth noticing here is not just what Cotra says, but that she says it at all. As a METR researcher, Cotra sits in an unusual position: METR is the external evaluator Anthropic invited in to assess whether its own RSP thresholds have been met. She's writing in public about the conclusions of that assessment — conclusions that implicate a threshold in Anthropic's own policy meant to trigger significantly more stringent safety requirements: strong model-weight security, universal logging and monitoring, robust alignment training, and technical measures against monitor subversion.

She explicitly argues the underlying evidence should be made public so outside researchers can verify the claims. That's an argument she can't make internally without also making it in a post like this one. Most of the key METR data that informs her argument is still redacted.

The concrete ask is narrow: publish the evidence. The implication is wider: if the evidence says what METR suggests it says, the gap between required safeguards and current safeguards is large, and the industry has less time to close it than it probably assumed.
