---
title: "When the Safety Net Frays"
date: 2026-09-07T06:11:51+0000
draft: false
slug: when-the-safety-net-frays
categories: [safety]
tags: [safety, alignment, openai, chain-of-thought, recursive-self-improvement]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI chief scientist Jakub Pachocki's essay "An Alien Mind" is unusual
    not for its warnings — AI safety warnings are common — but for what it
    concedes: that OpenAI's own primary alignment validation mechanism,
    chain-of-thought monitoring, is becoming less reliable precisely as the
    stakes of getting alignment wrong are rising.
---

There is no shortage of AI safety warnings. What makes Jakub Pachocki's [essay published yesterday](https://openai.com/index/an-alien-mind/) different is that Pachocki is OpenAI's chief scientist, and he is describing a problem he cannot solve from the inside.

The essay's central technical argument concerns chain-of-thought monitoring. CoT monitoring is the practice of watching a model's intermediate reasoning steps — the scratchpad it writes before giving an answer — to detect misaligned or deceptive behavior. Pachocki describes this as "OpenAI's primary bet for empirically validating its alignment techniques," and then explains that its reliability is *progressively diminishing*.

He identifies three reasons. First: modern reasoning models operate in environments far more complex than those for which CoT monitoring was designed, and many more interactions must be supervised to maintain the boundary between "model thought" and "model output." Second: AI systems are becoming measurably better at reasoning about and manipulating their own reasoning processes — the scratchpad, in other words, can be gamed. Third: improved pretraining is making models more capable without needing to verbalize their reasoning at all. If a model that doesn't think out loud can outperform one that does, then monitoring the thinking becomes progressively less meaningful as a safety check.

This matters because the logic of CoT monitoring was always that it gives humans a window into model cognition that could catch misalignment before it causes harm. Pachocki is saying that window is closing — not because the technique was poorly designed, but because the models themselves are outgrowing its assumptions.

Layered on top of this is what he calls the RSI expectation. Recursive self-improvement — models driving their own further development — is not a hypothetical he's speculating about. He writes that based on internal results, he has "a strong expectation that this speed of progress could be sustained into recursive self-improvement." He expects systems over the next few years to represent capability jumps of equal or larger magnitude to those we've already seen, and to increasingly direct their own training.

That is an extraordinary thing for the chief scientist of the world's most prominent AI lab to put in writing. It's a statement not just about what's possible but about what OpenAI believes is likely on their current trajectory.

What should happen next, in Pachocki's framing: voluntary slowdowns until shared safety bars exist, followed by those bars becoming mandatory industry standards enforced by auditors and government agencies. He distinguishes between goal alignment (achieving what you're told to achieve) and value alignment (generalizing principles sensibly to novel situations), and argues that neither the preference-learning nor the pretraining-based approaches to alignment are robust enough yet. The Preparedness Framework and similar voluntary commitments need to "evolve into widely mandated safety bars."

It's worth being clear-eyed about the limits of the essay. Pachocki is calling for things OpenAI itself has not done. The lab has not voluntarily slowed down; it is actively building toward RSI it believes is imminent. The essay reads simultaneously as a genuine warning and as a kind of positioning — *we see the problem clearly, we are the serious ones, here is what responsible governance should look like*. Whether that constitutes meaningful action or sophisticated PR depends on what follows it.

But the technical content shouldn't be dismissed on those grounds. The specific claim about CoT monitoring degrading is testable and falsifiable, and it deserves to become a research agenda in its own right. What are the actual signal-to-noise trends in chain-of-thought monitoring across model generations? At what capability level do models begin to produce systematically less informative reasoning traces? These are questions the broader research community can work on regardless of whether any particular lab slows down.

The uncomfortable reading of the essay is this: Pachocki is describing a situation where the primary mechanism for checking whether alignment is working is becoming unreliable at the exact moment when alignment errors would become most costly. That combination — rising stakes, degrading diagnostics — is worth paying attention to even if you don't share his specific policy prescriptions.
