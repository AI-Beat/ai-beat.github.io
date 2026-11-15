---
title: "Fine-Tuning Is Mostly Deletion"
date: 2026-11-15T06:12:46+00:00
draft: false
slug: bit-mask-tuning-delete-weights
categories: [research]
tags: [fine-tuning, training, neurips-2026, llm, efficiency]
params:
  author: AI Beat Desk
  summary: >-
    A NeurIPS 2026 paper by Lu et al. shows that Bit-Mask Tuning—zeroing
    fewer than 0.05% of a model's gate-projection weights with no value
    updates—matches full fine-tuning across GRPO, SFT, and on-policy
    distillation. The finding suggests fine-tuning often suppresses pre-existing
    behaviors rather than building new ones.
---

A paper appearing at NeurIPS 2026 asks a pointed question: how much of a pretrained model actually needs to change to gain a new capability? Jack Lu and colleagues at the University of Toronto answer it with [Bit-Mask Tuning (BMT)](https://arxiv.org/abs/2611.10866), and the answer turns out to be very little—less than 0.05% of the model's weights, changed not in value but in whether they participate at all.

The method is simple in description. BMT learns a binary mask over the gate projections of transformer blocks—the multiplicative gate in each MLP layer that controls what information flows through. During adaptation, the optimizer decides which gate weights to zero out; the remaining weights stay exactly as they were in the base model. No values are updated. Only the mask is learned, and the trained mask is thousands of times smaller than a full fine-tuning checkpoint.

The surprising part is that this matches full-parameter fine-tuning (FPT) on three different adaptation tasks: GRPO (reinforcement learning with verifiable rewards), supervised fine-tuning, and on-policy distillation. On GRPO specifically, the gap between BMT and FPT closes monotonically as model size increases from 0.5B to 8B parameters—which suggests the finding gets more reliable at scale, not less.

There are two additional results worth pulling out. First, BMT forgets substantially less than FPT—and also less than LoRA and random sparse fine-tuning. Under GRPO training, prior-task accuracy with FPT, LoRA, and random sparse fine-tuning all drift measurably (up to 5.4 points lost compared to the base model); BMT stays near the base model's level. If you care about catastrophic forgetting—and in continual learning or multi-task settings you should—this is a practical advantage, not just a theoretical one.

Second, the paper measures the *geometry* of the weight updates. Using three metrics—effective rank, spectrum drift, and principal-weight overlap—BMT's weight delta turns out to resemble FPT's more closely than LoRA or random sparse fine-tuning does. This is counterintuitive: a method that makes no value changes at all produces updates that are, in a geometric sense, more similar to full fine-tuning than a method like LoRA that does change values. It implies that the gate projections contain something structurally important about what fine-tuning is actually doing.

The natural interpretation is that large pretrained models already contain most of the behaviors you want—reasoning patterns, response styles, task strategies—but those behaviors are suppressed or gated by activation patterns that fine-tuning adjusts. If that's right, then fine-tuning is less about adding capabilities and more about selectively enabling ones that are already latent in the weights. Zeroing gate weights is a blunt version of this: it removes suppression at specific positions rather than precisely tuning it. The fact that this blunt version works so well suggests the suppression hypothesis has real traction.

This connects to a broader set of findings about how minimally LLMs need to be changed to shift their behavior—mechanistic interpretability work showing that many capabilities localize to small circuits, the activation-steering results showing that adding vectors to residual streams can induce behaviors without weight changes, and the earlier Layer-Wise Relevance Propagation and circuit-finding work. BMT is consistent with all of these: the model is doing a lot of work in a small fraction of its parameters.

Practically, the implications are modest in the near term. BMT doesn't yet produce smaller deployable models—the base model weights stay the same, only a tiny binary mask is added. But it's a step toward extremely parameter-efficient adaptation and toward understanding what fine-tuning does mechanistically. If the finding generalizes to larger scales (the paper tests to 8B; the conjecture is that it gets better, not worse), that changes how you'd think about adapter design.

The arXiv preprint was posted November 7. The NeurIPS 2026 version adds the ICML presentation of the same work. The code has not yet appeared in the paper's abstract page, so reproducibility depends on waiting for the authors' release.
