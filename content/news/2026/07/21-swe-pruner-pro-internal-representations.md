---
title: "The Agent Already Knows What's Worth Reading"
date: 2026-07-21T06:10:51+00:00
draft: false
slug: swe-pruner-pro-internal-representations
categories: [research]
tags: [coding-agents, context-pruning, interpretability, swe-bench, efficiency]
params:
  author: AI Beat Desk
  summary: >-
    SWE-Pruner Pro, submitted to arXiv on July 20, shows that coding LLMs encode
    relevance signals for their own tool outputs inside their residual stream —
    and a lightweight head reading those activations can prune 39% of tokens
    while actually improving SWE-Bench Verified performance by 3.8%.
---

There's a persistent engineering problem in coding agents: tool outputs — file contents, shell outputs, search results — accumulate fast, and context windows fill with noise that the model has to sort through on every forward pass. The original [SWE-Pruner](https://arxiv.org/abs/2601.16746) attacked this with a task-aware external classifier trained to strip irrelevant lines before they entered the context. It worked, but it added a separate component with its own latency and training budget.

[SWE-Pruner Pro](https://arxiv.org/abs/2607.18213), submitted July 20, asks a simpler question: does the model already know? The answer turns out to be yes.

The paper's central finding is that when a coding LLM reads a tool output, its residual stream at the final token of each line encodes whether that line matters for the current task. This isn't a post-hoc interpretation — they verify it directly, training a lightweight probe on those internal representations that outputs a keep-or-prune binary label per line. The probe is small enough that its inference overhead is bounded, and because it attaches inside the agent's own forward pass rather than running separately, it has access to full task context without being a separate model.

A length-aware embedding is added per tool output to handle the fact that pruning decisions depend on how long an output is — short files behave differently from 10,000-line diffs. Beyond that, the mechanism is straightforward: during an agent turn, lines of tool output get labeled by the probe and the low-relevance ones are dropped before they re-enter the context for the next step.

The numbers on [MiMo-V2-Flash](https://huggingface.co/XiaomiMiMo/MiMo-V2-Flash) are strong. Prompt and completion tokens each drop by up to 39% — both matter, since completions are what you're paying for at most inference providers. On SWE-Bench Verified, task performance goes up 3.8%; on Oolong (a long-context benchmark) it gains 2.2 points. The fact that pruning improves accuracy, not just efficiency, is the interesting part. The model is getting less noise, not fewer resources; the task becomes easier when the context is tighter.

The interpretability angle here is worth pausing on. The finding that a model encodes relevance signals in its own activations is consistent with what we know from mechanistic interpretability work on factual recall and circuit analysis — models develop internal structure that the residual stream carries forward even when it's not explicitly asked about. What SWE-Pruner Pro adds is a practical use: you can use that structure to filter the model's own inputs.

This is meaningfully different from something like a KV-cache compression technique, which trims attention weights after the fact. Here the pruning happens at the input level, before those tokens ever enter the context. It's also different from learned compression approaches that train a separate small model; the probe is anchored to the coding agent's own representations and has no existence outside them.

One open question is generalization: the experiments use MiMo-V2-Flash and a second unnamed open-weight backbone. Whether the same lightweight head transfers across very different model families, or whether each new backbone needs its own probe training, will determine how practical this is in deployment. The paper doesn't claim universality. But as an illustration of a general principle — that models' internal representations are a resource worth using to guide their own inputs — it's a clean result.
