---
title: "Against Superposition"
date: 2026-10-16T06:10:00+00:00
draft: false
slug: against-superposition-goodfire
categories: [research]
tags: [interpretability, mechanistic-interpretability, research]
params:
  author: AI Beat Desk
  summary: >-
    Nick Cammarata's essay at Goodfire argues the interpretability field's
    central premise — that neural networks are opaque because of superposition
    and polysemanticity — is largely wrong, and that most neurons in large
    models are already interpretable, with ConvNeXt ablation experiments and
    a Qwen3-32B audit as evidence.
---

The mechanistic interpretability field has a founding assumption: neural networks are hard to read because of superposition. The story goes that models pack more features than they have neurons, so each neuron ends up encoding multiple unrelated concepts in a compressed, overlapping way. Sparse autoencoders (SAEs) became the dominant tool precisely because of this — you first have to decompose the representation into something interpretable before you can study it. Nick Cammarata at Goodfire published a working hypothesis this week that challenges this directly: [neural networks are already clean](https://already-clean.goodfire.com/).

The argument starts with a reread of the canonical examples. The famous "cat and car" neuron in InceptionV1 — cited for years as evidence of polysemanticity — is better described, Cammarata argues, as a V-shape detector that responds to the structural geometry shared by cat ears and car roofs. What looked like two unrelated concepts packed into one neuron was a generalization the researchers didn't recognize. The essay calls this "recursive generalization": neurons develop related offshoots during training, not orthogonal superimposed concepts. A neuron that handles several visually adjacent ideas isn't polysemantic in the concerning sense — it's just broadly tuned.

This distinction matters because the practical implications diverge. If neurons are genuinely polysemantic in the superposition sense, SAEs are essentially mandatory — you need to decompose the representation before you can say anything about it. If neurons generalize the way Cammarata describes, you can study them directly, and the SAE layer may be adding structure that isn't native to the model.

The evidence he presents comes mainly from ConvNeXt experiments. When he removed the weakest 75% of activations, accuracy barely moved (72% to 71%). When he removed the strongest 7%, it collapsed to 4%. His read: weak activations are generalization noise, not compressed hidden features. If superposition were the primary mechanism, weak activations should carry real signal, just in a distributed form. An SAE analysis of the same model found that roughly 88% of SAE activity mapped onto individual neurons, and removing the unmatched SAE directions — the directions SAEs found that neurons didn't cover — left accuracy unchanged.

For LLMs, a preliminary audit of Qwen3-32B found approximately 90% of neuron poles with a single clear job, about 7% with two related jobs, and 2–4% genuinely unexplained. Cammarata also built attribution graphs — the kind Anthropic has published for smaller models — using raw neurons rather than SAE features, for both Qwen3-32B and Kimi K3. He describes the Kimi K3 graph as possibly the first attribution graph of a multi-trillion-parameter model built from neurons directly, and there's a companion paper on it forthcoming.

The standard counterargument applies: this is a working hypothesis, the essay is careful to say so, and the audits are incomplete. The ConvNeXt experiments are on a much smaller scale than the LLMs being discussed. Cammarata notes explicitly that his neuron percentages "depend heavily on thresholds and are not precise." The Kimi K3 graph hasn't been peer-reviewed.

But the direction is interesting precisely because it would simplify things considerably. A recurring critique of SAE-based interpretability is that SAEs impose their own structure on a model, and it's not obvious whether the features they recover reflect the model's actual representations or the particular decomposition the SAE learned. If most of the signal is already in the neurons, a lot of that concern evaporates, and direct neuron analysis at scale becomes more tractable than the field has assumed.

The piece is long — labeled a "65-minute read" — and goes into neuron ablation methodology, visualization choices, and negative pole analysis in real depth. Cammarata also reports that adding residual connections and LayerNorm to InceptionV1 made its last layer substantially more interpretable, with about 99% of neurons in a comparable ConvNeXt model being easy to read. That's a practical note: part of what makes networks clean may be design choices we've already made, not a property that needs to be excavated with additional tooling.

