---
title: "The Geometry of Convergence"
date: 2026-11-02T06:15:00+00:00
draft: false
slug: platonic-representation-jepa
categories: [research]
tags: [research, representation-learning, multimodal, theory, LeCun]
params:
  author: AI Beat Desk
  summary: >-
    Balestriero and LeCun formally prove the Platonic Representation Hypothesis
    for JEPAs: models trained separately on any modality learn the same
    representation up to rotation, provided they use good positive pairs.
    The result reframes next-token prediction as a JEPA with structurally
    bad views, explains the slow alignment of language models with vision,
    and shows a 109M text JEPA outperforms every LLM tested up to 72B
    parameters on cross-modal tasks.
---

The [Platonic Representation Hypothesis](https://arxiv.org/abs/2405.07987), proposed in 2024, made a striking empirical observation: large models trained on different modalities seem to converge toward a shared representation of the world. Vision transformers, language models, audio encoders — at sufficient scale, their internal geometries grow more similar. The hypothesis was intuitively appealing but didn't come with a proof, and "they converge" is a different claim from "they are guaranteed to converge to the same thing."

A new [paper by Randall Balestriero and Yann LeCun](https://arxiv.org/abs/2610.39302) now supplies the proof — but only for a specific architecture: Joint-Embedding Predictive Architectures (JEPAs). The proof also clarifies exactly what determines whether convergence happens, and the answer is more about training data structure than scale.

## The Platonic Representation Theorem

The key result: JEPAs trained on different modalities with "good views" — where positive pairs are two views of the same scene — learn the same representation up to a linear rotation. When the views are "bad" (not truly two windows onto the same underlying event), the representations can still be bounded in their divergence, but no convergence guarantee holds.

That rotation-equivalence is important. It means independently trained encoders aren't the same vector space, but they're the same space up to a change of basis. Finding that rotation is cheap — a linear solve on aligned samples — and the paper demonstrates this by stitching together separately trained image and text encoders into a working image captioner (CIDEr 0.838 on COCO, no joint training required).

## Where does next-token prediction fit?

This is the most interesting framing in the paper. Standard language modeling can be cast as a JEPA where the positive pairs are adjacent segments of the same document: the left context and the right context are "views" of the same underlying content. But adjacent text fragments are structurally weaker views than two camera angles of the same scene — the information overlap is lower, the noise is higher, and the shared signal is diluted through many possible continuations.

The authors call these "bad views," and their theory predicts that language models should converge toward visual representations only slowly, through scale. The empirical evidence fits: language models do align with vision encoders, but the relationship looks like a scaling law rather than a structural guarantee, and large models still trail behind encoders with better-structured objectives.

## The sample efficiency gap

Training a JEPA text encoder with good views — pairing two independently sampled summaries of the same document, for instance — requires [about 42 times fewer samples](https://arxiv.org/abs/2610.39302) to reach the same cross-modal alignment as next-token prediction on the same corpus. The paper tests a 109M-parameter text JEPA and finds it aligns better with DINOv2 and CLIP image encoders than every language model evaluated, up to 72B parameters. On a pair-free zero-shot benchmark the text JEPA reaches 64% top-5 accuracy on ImageNet-100 versus at most 56% for comparably large language models.

The upshot is that the quality of positive pairs is not a training trick — it's the structural variable that determines whether you get a platonic encoder or just an approximation to one.

## What this implies

The result changes how you should think about building multimodal systems. If you want modality-agnostic representations, the path isn't "train a bigger language model" or "scale up contrastive learning." It's "improve your views" — find a way to construct positive pairs that are genuinely two renderings of the same scene.

It also opens a low-cost route to composing existing encoders. If two JEPA encoders trained independently both approximate the platonic representation, you can find the rotation between them with aligned examples and skip joint training entirely. This works across modality boundaries by construction, not by luck.

The theoretical contribution here is not just formalizing something people already believed. It reframes a decade of empirical observation as a geometric inevitability — conditional on architecture and data structure choices that are now clearly spelled out.
