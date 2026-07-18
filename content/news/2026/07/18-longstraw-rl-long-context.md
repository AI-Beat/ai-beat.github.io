---
title: "Training Agents on What They Actually Read"
date: 2026-07-18T06:08:42+00:00
draft: false
slug: longstraw-rl-long-context
categories: [research]
tags: [rl, training, long-context, agents]
params:
  author: AI Beat Desk
  summary: >-
    LongStraw extends reinforcement learning post-training to 2.1M-token
    contexts on eight H20 GPUs, closing the awkward gap between what models
    can read at inference and what they can be trained on via RL—a gap that
    matters increasingly as agents accumulate long histories of tool calls and
    observations.
---

There is an uncomfortable mismatch at the center of agent training right now: models at inference time routinely operate over million-token contexts, but RL post-training—the stage that actually teaches models to make better sequential decisions—typically caps out at 256K tokens. The rest is cut off. The model learns to reason over evidence it can read at deployment time, but the training signal only covers the first quarter-million tokens of what it will eventually face.

For most narrow applications this doesn't matter much. But for agents—which accumulate long histories of tool calls, environment observations, partial results, and error messages—it matters a lot. The states that require the most learning are often the ones with the longest context.

[LongStraw](https://arxiv.org/abs/2607.14952), out of Mind Lab and submitted to arXiv on July 16, attacks this problem directly. The core contribution is an execution framework that makes million-token RL training viable within a fixed GPU budget—specifically 2.1M tokens on eight H20 GPUs, with stress tests pushing to 4.46M positions across 32 H20s.

The technical approach is built on Group Relative Policy Optimization (GRPO) and addresses the three main memory bottlenecks that make long-context RL expensive. First: shared prompts within a group (multiple completions for the same context) are evaluated once without gradient computation and cached, rather than forwarded redundantly through the model for each completion. Second: only the model states strictly necessary for the backward pass are retained, rather than keeping full activations for the entire sequence. Third: instead of processing all response branches in parallel (which would require holding them all in memory simultaneously), responses are replayed sequentially during the gradient step.

None of these tricks is exotic in isolation. The contribution is combining them into an architecture-aware stack that scales—and validating it end-to-end on hybrid recurrent-attention and compressed mixture-of-experts model architectures, which are the kinds of models designed to handle long sequences efficiently.

The practical significance is that RL post-training and inference can now operate at comparable context lengths for the first time on accessible hardware. Previously, if you were training an agent to handle multi-step tool use, you'd either have to truncate its history (losing signal from earlier in the trajectory) or run on a GPU cluster large enough to hold the full activations in memory (expensive, slow). LongStraw offers a third option: optimize memory use intelligently enough that eight H20s can handle what previously required much more hardware.

The paper's main experiments target hybrid recurrent-attention models, which are a natural fit for long contexts because they handle long-range dependencies more efficiently than pure attention. But the framework is architecture-general, and the sequential response replay in particular is an execution pattern that should port without much modification.

This kind of infrastructure work tends to get less attention than new model releases or benchmark jumps, but it's often where the durable progress happens. The gap between what agents can read and what they can be trained on was a real bottleneck. Papers like this are how it gets removed, quietly, one compute constraint at a time.
