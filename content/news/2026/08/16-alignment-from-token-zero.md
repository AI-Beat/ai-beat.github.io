---
title: "Alignment Before the Fine-Tune"
date: 2026-08-16T06:45:00+00:00
draft: false
slug: alignment-from-token-zero
categories: [research]
tags: [alignment, pretraining, safety, llm]
params:
  author: AI Beat Desk
  summary: >-
    A paper from EPFL's dlab and the MATS alignment program proposes threading
    value-laden reflections through 10% of pretraining documents, showing a
    63% reduction in adversarial attack success — but only when post-training
    uses precisely matching persona-binding templates. The brittleness is the
    most revealing finding.
---

The standard story about AI alignment goes: you pretrain a model on a large corpus, then apply RLHF or constitutional AI or some variant of preference learning to instill values into the resulting model. The pretrained weights are treated as a foundation to be shaped, not a place to put values.

A [new paper from researchers at EPFL's dlab and the MATS alignment program](https://arxiv.org/abs/2608.13482) challenges that assumption. Their method, Synthetic Persona Pretraining (SPP), threads value-laden reflections through 10% of pretraining documents before training starts. The idea: rather than adding alignment as a layer above the base model, bake it in from token zero.

The mechanics are straightforward. Take a standard pretraining corpus. For 10% of documents, append a first-person reflection that engages with the document's content through an ethical lens — commentary grounded in a six-domain value constitution, separated from the document by an assistant token. The model trains on these pairs with ordinary cross-entropy loss. The assistant then "sees" examples of the kind of reasoning the target persona would apply, distributed throughout pretraining, rather than encountering it for the first time in fine-tuning.

The results on 3B parameter models trained on 500B tokens show a 63% reduction in attack success rates across five adversarial benchmarks compared to baseline models with identical post-training. Constitution following improves, out-of-distribution moral dilemma performance improves, and capability benchmarks are essentially unaffected. The finding that the advantage "increases with pretraining budget" is the kind of scaling-law-adjacent result that tends to attract attention.

Two things make the paper worth reading carefully beyond the headline number.

First, the timing effect. Models trained with SPP reflections only in the final portion of pretraining performed substantially worse than models that received reflections from the beginning. The authors describe this as early intervention mattering — values installed at token zero outperform values installed midtraining on identical volumes of reflection data. This implies the effect isn't purely about quantity of alignment signal; it's about when during training the model encounters it, presumably because earlier exposure shapes how the representations develop.

Second, and more interesting: **persona binding is brittle**. The safety improvements from SPP don't automatically transfer through post-training. If the supervised fine-tuning phase uses a different assistant token format than the one used in pretraining, the benefits disappear entirely. The pretraining-installed persona exists at a specific representational "address" that post-training must connect to with matching templates. Wrong token, wrong template — alignment benefit erased.

This has two implications. The practical one: anyone attempting to reproduce or build on SPP needs to keep the assistant token format consistent through the entire training pipeline, which creates a fragile dependency between pretraining decisions and fine-tuning infrastructure. Organizations that fine-tune base models they didn't pretrain would have no way to benefit from SPP unless the pretraining provider used SPP and documented the exact persona-binding template.

The more conceptually interesting implication is what brittleness suggests about where alignment lives in the model. If the safety signal inhabits a narrow representational neighborhood that's easy to miss with a slightly different template, it's an interpretability target: you can potentially locate it, probe it, and understand why it activates. It's also a vulnerability: white-box access plus knowledge of the expected token format gives you a way to route around the safety installation entirely. The paper acknowledges this without resolving it.

The setup — EPFL dlab and the MATS mentorship program — puts this in the academic alignment research category rather than a lab's production safety work. The 3B parameter scale is modest by current standards, and scaling behavior to 70B or 200B models is an open question. But the pretraining-time intervention idea is clean enough that it will probably show up in production pipelines in some form, if it hasn't already. The baseline assumption that pretraining is value-neutral and alignment is a post-hoc operation may be starting to erode.
