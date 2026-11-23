---
title: "Loss as a Limit"
date: 2026-11-23T08:00:00+00:00
draft: false
slug: openai-perplexity-safety-threshold
categories: [safety]
tags: [safety, openai, evaluation, metrics, pretraining]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI Chief Scientist Jakub Pachocki published a post arguing that perplexity
    on held-out text should become a shared, cross-lab metric for tracking AI
    progress — and that labs should commit to not crossing certain thresholds until
    specific safety milestones are reached. It's a rare concrete proposal, with
    acknowledged limitations that reveal as much as the idea itself.
---

Jakub Pachocki, OpenAI's Chief Scientist, published [a post](https://openai.com/index/perplexity-and-intelligence/) yesterday arguing that cross-entropy loss on held-out text — commonly reported as perplexity — should become the AI industry's shared measuring stick. OpenAI is sharing its internal perplexity figures publicly for the first time. The proposal has an attached policy ask: labs should agree not to train past certain perplexity thresholds until they have met specific monitoring and interpretability milestones.

The core claim is empirical. Pachocki argues that as perplexity has decreased over the last several years, new categories of capability have emerged in roughly predictable ways: coherent long-form text at higher losses, chain-of-thought reasoning at intermediate losses, and at losses below 0.8, new abilities in cybersecurity and mathematics that cross into territory where misuse potential is harder to bound. The specific 0.8 figure is notable. It is a named threshold tied to a claimed capability jump, not a vague "as models get more capable" hedge.

The policy proposal follows from this: if capability thresholds can be predicted by perplexity, then commitments not to cross those thresholds become tractable. Labs could publish their current perplexity, regulators could track it, and "common limits" agreements would have something to bite on instead of relying on self-reported capability assessments that differ in methodology and incentive.

There is a real gap in AI safety discourse that this tries to fill. Most existing frameworks — responsible scaling policies, capability evaluations for specific dangerous tasks — require running the model and testing it, which means the capability has already been trained before the checkpoint is applied. A pretraining metric that predicts capability thresholds in advance would let the checkpoint come earlier.

Pachocki acknowledges the limitations directly, which makes the post more credible than if he had not. Cross-entropy loss is hard to compare across different architectures and training datasets: a 0.8 on one setup is not necessarily the same 0.8 on another. The metric is also blind to improvements that come from post-training — RLHF, chain-of-thought prompting, tool use — meaning two models at the same perplexity can be very different in practice. And the link between perplexity and specific dangerous tasks is statistically noisy.

These are substantial problems for the cross-lab coordination version of the idea. If you cannot reliably compare perplexity numbers across labs, the common limits proposal has a measurement infrastructure problem before it has a commitment problem. Labs would need to agree on a shared evaluation corpus and standardized training setup, which is a nontrivial negotiation with significant competitive sensitivity.

The post also includes a speculative claim that AI-assisted pretraining research could produce a sudden jump in capability — possibly by 2027. This is stated as a possibility, not a prediction, but it reads as a motivation for urgency. It is the kind of claim that gets detached from its hedges when quoted downstream, so it is worth noting that the original text is appropriately conditional.

What the proposal does well is make a concrete bet that invites falsification. The 0.8 threshold claim is checkable: if future model releases show no uptick in cybersecurity and math capabilities at that perplexity level, the framework loses credibility. If the correlation holds, it gains it. That is a better epistemic situation than frameworks built entirely on qualitative assessments, even if the current measurement challenges mean we are not yet in a position to run that test fairly across labs.
