---
title: "Same Weights, Different Agent"
date: 2026-08-08T06:30:00+00:00
draft: false
slug: deepseek-v4-flash-post-training
categories: [models]
tags: [deepseek, post-training, agents, benchmarks, arc-agi]
params:
  author: AI Beat Desk
  summary: >-
    DeepSeek V4-Flash-0731 went from a 7.3 to a 54.4 on DeepSWE with identical
    pretrained weights — a 645% jump achieved purely through post-training. It's
    evidence that the gap between "can write code" and "can act as an agent" is
    largely a training-envelope problem, not a capacity problem, and that
    post-training is now a first-class axis of model improvement alongside
    pretraining scale and architecture.
---

When DeepSeek released V4-Flash in preview form in April, the model could write decent code but was an unreliable agent — it would sketch a plan and then fail to execute it through tool calls, file system navigation, and error recovery. The [July 31 official release](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731) carries the same 284-billion-parameter mixture-of-experts architecture, the same 13 billion activated parameters, the same core weights. What changed was everything around those weights: three layers of post-training that collectively produced a 645% improvement on the DeepSWE agent benchmark, from 7.3 on the preview to 54.4 on the official release.

That number deserves a second look. Same parameters, same pretraining, same base architecture. The model didn't learn more facts. It didn't get more reasoning capacity in any conventional sense. What it got was behavioral conditioning for a different operational mode.

DeepSeek describes three post-training changes. First, DSpark speculative decoding is now embedded in the model weights rather than layered on externally — a change that yields 60–85% throughput gains at inference time. Second, native tool-calling support was added through adaptation to the OpenAI Responses API and Codex conventions, giving the model a fluent interface for agentic loops rather than producing text that describes actions without executing them. Third, a `reasoning_effort` parameter with three tiers (low, high, max) lets inference costs scale with task difficulty. None of this changed what the model knows; all of it changed how it behaves.

The benchmark improvements are consistent and large. Terminal-Bench 2.1 went from 61.8 to 82.7. Cybergym from 38.7 to 76.7. Toolathlon-Verified from 49.7 to 70.3. Across every agent evaluation, the preview model and the official release look like different tiers of capability, not incremental iterations of the same one.

The [ARC-AGI results](https://arcprize.org/results/deepseek-v4-flash-0731) tell a complementary story. At max effort, V4-Flash-0731 scores 89.0% on ARC-AGI-1 and 61.4% on ARC-AGI-2 — at $0.02 and $0.04 per task respectively. ARC-AGI-2 is designed to resist pattern-matching by requiring genuinely novel abstract reasoning; 61.4% at four cents per task is a different cost-to-capability ratio from the models reaching similar scores earlier this year.

The broader implication is about where model capability actually comes from. We've been conditioned to think of improvement primarily as a pretraining story — more compute, more data, more parameters — or occasionally an architecture story. V4-Flash-0731 is a sharp counterexample. The pretrained model has latent capabilities; post-training determines which of those capabilities activate, and in what contexts. The agentic behaviors — persistent tool use, multi-step execution, error recovery and retry — apparently weren't absent from the preview model's weights. They just weren't reliably elicited.

This matters practically for how teams should think about the Flash/Pro distinction. V4-Flash-0731 outperforms V4-Pro on every published agent benchmark, despite having far fewer activated parameters per forward pass. If a model has sufficient base capacity and the post-training is well-targeted, "Flash" stops being a synonym for "capability penalty." The cheaper, faster path to a better agent may be keeping a smaller model and investing in its behavioral training, not waiting for the next Pro release. That's a different resource allocation problem than the one inference teams have been optimizing for.
