---
title: "The Audit Can't See It"
date: 2026-10-13T06:11:06+00:00
draft: false
slug: capability-hiding-neural-backdoors
categories: [safety]
tags: [safety, evaluation, cryptography, backdoors]
params:
  author: AI Beat Desk
  summary: >-
    A NeurIPS 2026 paper embeds encrypted neural networks inside model weights
    using cryptographic digital lockers. The hidden capability survives every
    known elicitation method — fine-tuning, weight noising, gradient search,
    adversarial training — placing a provable limit on what capability
    evaluations can guarantee.
---

Capability evaluations rest on a quiet assumption: that if a dangerous capability is
in a model, someone trying hard enough will find it. Fine-tune on examples of the
behavior. Search gradients for triggers. Inject weight noise to destabilize
sandbagging. If something is there, these techniques should surface it.

[A paper accepted at NeurIPS 2026](https://arxiv.org/abs/2610.13928) by Draguns,
Motwani, Douglas, and Schroeder de Witt shows this assumption has a hard ceiling —
and that ceiling is cryptographic.

## The construction

The core idea is to embed an encrypted secondary neural network inside the weights
of a host model. The hidden network stays dormant until the model receives a specific
trigger; without the trigger, it never activates and its presence is indistinguishable
from normal weight variation.

The security guarantee comes from *digital lockers*, a cryptographic primitive that
lets you bind a function to a key such that evaluating the function requires the key
and cannot be reversed without it. Even with full access to the model weights — which
evaluators typically don't have — reconstructing the trigger or the hidden function
is computationally equivalent to breaking the underlying one-way function.

The authors prove this under standard cryptographic assumptions and back it with a
PyTorch proof of concept on chemical reaction prediction, used as a proxy for
CBRN-relevant capabilities. They then ran the hidden capability through every
elicitation method currently in use: fine-tuning on demonstrations of the target
behavior, weight noising (a technique specifically designed to break sandbagging),
gradient-based trigger search, and latent adversarial training. None of them worked.

Prior work in this space could hide fixed strings. This construction hides arbitrary
computations. That's a meaningful step: it moves from "a model can suppress an answer"
to "a model can run an entirely different program depending on who is asking."

## What this actually means

The practical threat model requires a motivated adversary with the ability to modify
model weights before or during training. That's not a casual attacker — it's a
rogue insider at a training lab, a supply chain compromise, or an organization that
wants to certify a model it knows has hidden functionality.

The paper is careful not to overstate. The proof of concept is small-scale, the
guarantees rest on computational hardness assumptions that cryptographers take
seriously but which are not unconditional, and the construction hasn't been shown to
scale cleanly to frontier-sized models. The authors explicitly release their
implementation to support defense research.

Still, the structural point is worth sitting with. Current AI governance frameworks —
whether voluntary or regulatory — treat capability evaluations as a meaningful
check on what a model can do. If an adversarial actor can embed capabilities that are
cryptographically hidden from those evaluations, the check stops being meaningful
for the specific case you most care about: the deployment of a model whose developers
want to hide what it does.

This is not a new theoretical concern. Sandbagging — models strategically
underperforming on evaluations — has been studied for a few years now. What's new
here is the strength of the guarantee. Earlier sandbagging relied on the model
learning to hold back during evaluation without any hard security property; elicitation
techniques could often break it. Digital-locker backdoors are a category above that:
they remain hidden unless the evaluator holds the trigger, and there's no known way
to find the trigger without it.

The appropriate response probably isn't to abandon capability evaluations — they're
still useful against honest-but-mistaken training and against models that sandbag
without cryptographic support. The response is to be clear about the threat model
they actually cover and to stop treating a clean evaluation as a guarantee against
deliberate evasion.
