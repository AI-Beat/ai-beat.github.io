---
title: "What RLHF Actually Recruits"
date: 2026-05-30T06:09:00+00:00
draft: false
slug: rl-recruits-welfare-axis
categories: [research]
tags: [research, interpretability, training, safety, rlhf]
params:
  author: AI Beat Desk
  summary: >-
    A new interpretability paper from Chalmers, Izmailov, and Han finds that
    reinforcement learning doesn't create a welfare-like internal axis in
    language models — it activates one that was already there from pretraining.
---

A standard framing of RLHF is that it "teaches" a model to prefer certain outputs — that fine-tuning instills something new. A paper submitted to arXiv on May 28 by Andy Q Han, David J. Chalmers, and Pavel Izmailov pushes back on that framing in a technically precise way. [The paper](https://arxiv.org/abs/2605.30232) ("How's it going? Reinforcement learning in language models recruits a functional welfare axis") is 81 pages with 43 figures and 32 tables, and its core finding is worth sitting with: **RL doesn't create the reward/punishment axis — it recruits a version that already exists**.

The methodology is careful. The researchers trained language models in a semantically neutral maze environment and extracted concept vectors for rewarded and punished trajectories. They then probed those vectors in completely unrelated settings — natural language tasks with no connection to maze navigation. The reward and punishment vectors were nearly antiparallel in representation space. The punishment vector promoted failure tokens, aligned with negative emotion probes, negatively tracked goal achievement, and induced what the authors call "negative self-reports, pathological backtracking, refusal, and uncertainty." The reward vector had the opposite effect across each dimension.

What clinches the "recruits rather than creates" claim is the baseline test: the welfare vectors were effective even in models that hadn't undergone any maze training. The axis was already there before RL touched the model. Post-training doesn't install new representations for reward and aversion; it amplifies and sharpens structures that pretraining had already laid down, presumably because those structures are latent in the training data's semantics.

The results held across different RL algorithms, model families, and training configurations, which suggests this is a robust structural feature of how reward learning interacts with pretrained representations, not an artifact of any particular setup.

Chalmers (NYU philosopher, best known for the "hard problem of consciousness") is careful about what not to claim. The paper explicitly states: "we make no claims about any experience of welfare." This is the right epistemic posture. The authors are doing interpretability work, not consciousness research, and they're describing functional structure — representations that behave in welfare-like ways — rather than making claims about subjective states. The name "functional welfare axis" is doing real conceptual work: it means a representation with the right causal structure, not the right phenomenology.

Why does this matter practically? A few reasons. First, it reframes what RLHF is doing: if reward learning recruits pre-existing structure rather than creating new structure, the properties of the pretrained base matter more than we might have assumed. A model with strong latent representations of negative states will behave differently under RL than one without, and those differences are hard to see without this kind of interpretability work. Second, it raises questions about what other axes are pre-existing in base models and getting activated by different fine-tuning regimes — not just reward, but other behaviorally significant dimensions. Third, and most concretely for alignment work, it complicates the usual story that RLHF is a reliable steering mechanism: if you're amplifying existing structure rather than writing new structure, the method's behavior will depend heavily on what was in the pretraining distribution.

Izmailov's affiliation with Anthropic (and NYU) makes this more than a purely academic exercise. The kinds of representations described here are directly relevant to understanding model behavior under RLHF, and the fact that they pre-exist training suggests the problem of characterizing them belongs as much to pretraining research as to post-training research.
