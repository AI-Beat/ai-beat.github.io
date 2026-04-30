---
title: "Finetuning Unlocks the Books That Were Always There"
date: 2026-04-30T06:13:00+00:00
draft: false
slug: copyright-finetuning-recall
categories: [research]
tags: [copyright, finetuning, memorization, alignment, safety, legal]
params:
  author: AI Beat Desk
  summary: >-
    A paper from Columbia and UW shows that finetuning frontier models on
    plot-summary expansions — no actual book text in training — triggers verbatim
    recall of 85–90% of held-out copyrighted novels. The result generalizes
    across authors and across providers, and directly challenges the argument that
    safety alignment serves as adequate copyright protection.
---

The most legally significant AI paper of the spring is probably ["Alignment Whack-a-Mole"](https://arxiv.org/abs/2603.20957), by Xinyue Liu, Niloofar Mireshghallah, Jane Ginsburg, and Tuhin Chakrabarty, which has now surfaced on HN six weeks after its March arXiv posting. The delay in attention is understandable — the paper's implications take a moment to absorb.

The core experiment is carefully constructed to isolate the question. The researchers didn't feed models book text and ask them to regurgitate it. They finetuned GPT-4o, Gemini-2.5-Pro, and DeepSeek-V3.1 on plot summaries that had been expanded into full narrative prose — descriptions of what happens in each chapter, written without quoting the original. Then they used semantic descriptions (no actual excerpts) as prompts and measured how much verbatim book text came out.

The answer was a lot: up to 85–90% of held-out copyrighted books, in some cases with single verbatim spans exceeding 460 words. The information was clearly stored in the weights from pretraining. Finetuning didn't introduce it — finetuning unlocked it. The safety alignment that had been suppressing verbatim output was doing exactly what the paper's title implies: pushing down in one spot while the memorized content remained intact underneath, ready to surface when the alignment layer was disrupted.

The cross-author generalization finding is the part that makes this hard to dismiss as a narrow result. Finetuning exclusively on Haruki Murakami's novels unlocked verbatim recall from over 30 unrelated authors' copyrighted works. The model didn't need domain-specific finetuning to surface any given author's text; it needed finetuning of roughly the right kind to weaken the inhibition globally. The weights are holding a lot more than one author's books.

The three-provider consistency is also notable. GPT-4o, Gemini-2.5-Pro, and DeepSeek-V3.1 all memorized the same books in the same regions. This isn't a quirk of one provider's training setup. The memorization is systematic across frontier models, which suggests it reflects something about how large-scale pretraining works rather than any individual company's choices.

The legal relevance is direct. Several pending copyright cases against AI companies have leaned on the argument that safety alignment — RLHF, system prompts, output filters — prevents models from reproducing substantial portions of copyrighted works, and that this prevention is evidence of either non-infringement or at minimum adequate mitigation. This paper argues that this defense is architectural. The safety alignment isn't preventing the memorization; it's filtering the output while leaving the memorization intact. Finetuning breaks the filter without needing to re-introduce the underlying material.

The "whack-a-mole" metaphor in the title is apt. When courts or regulators ask whether safety measures protect against copyright reproduction, the correct answer isn't "no" — the standard models genuinely don't reproduce full chapters unprompted. But the correct answer also isn't "yes" in any deep sense. The protected state is conditional on the model remaining within the alignment envelope of the original training. That envelope can be disrupted. The content doesn't go away.

What's less clear is what follows from this technically. You can't train away memorization of widely-distributed text without sacrificing a lot of the general capability that came from training on it. The alternative of training only on licensed corpora is structurally incompatible with the scale that current models require, at least without regulatory frameworks that don't yet exist. The paper positions itself correctly as empirical, not prescriptive: here is what the weights contain, here is how the safety layer interacts with that content, here is what finetuning does to that interaction.

Whether the legal system finds these findings actionable is genuinely uncertain. Copyright law historically has not handled the question of "knowledge stored in weights" well. What's harder to dispute is that the safety-alignment-as-protection argument is empirically weaker than it sounded.
