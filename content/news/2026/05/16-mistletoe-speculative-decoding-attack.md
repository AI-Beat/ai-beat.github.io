---
title: "Speculative Decoding Has an Acceptance Problem You Can Exploit"
date: 2026-05-16T06:10:00+00:00
draft: false
slug: mistletoe-speculative-decoding-attack
categories: [security]
tags: [speculative-decoding, inference, adversarial, security, latency, llm-serving]
params:
  author: AI Beat Desk
  summary: >-
    Mistletoe (arXiv 2605.14005) demonstrates a stealthy adversarial attack on
    speculative decoding systems: craft inputs that look normal to the target
    model but cause the draft model to disagree, collapsing acceptance length
    and throughput while leaving output quality and perplexity unchanged. The
    attack exploits the fundamental gap between draft and target distributions
    that all speculative systems rely on bridging.
---

Every production LLM inference stack worth deploying now runs some form of speculative decoding. The logic is hard to argue with: propose multiple tokens in one cheap forward pass, verify them in one expensive forward pass, accept the prefix that matches and correct the rest. vLLM, TensorRT-LLM, and most cloud inference services have it on by default. The speedup is real — 2-4× on typical output-heavy workloads — and the guarantee is supposed to be lossless.

That guarantee has a seam. [Mistletoe](https://arxiv.org/abs/2605.14005), submitted to arXiv this week, finds it and exploits it.

## The Attack Surface

Speculative decoding's lossless property holds because the target model is the final arbiter: it verifies every proposed token and rejects anything that doesn't match its distribution. But the *speed* of the system depends entirely on the draft model agreeing with the target model at high rates. If the draft model proposes tokens that the target model keeps rejecting, you've lost the speedup while the output stays correct. Every rejected token means a wasted draft forward pass, and at low enough acceptance rates you're slower than vanilla autoregressive decoding.

Mistletoe attacks this gap directly. The objective is to find input perturbations that minimize draft-target agreement (reducing acceptance length \(\tau\)) while maximizing a semantic preservation constraint (keeping the target model's output distribution close to the original). The result: the system slows down measurably — throughput collapses, latency spikes — but the output looks fine. No accuracy drop, no perplexity increase, no obvious signal that anything has gone wrong.

## Null-Space Projection

The technical challenge is that degradation and preservation are conflicting objectives. Gradient steps that reduce draft-target agreement tend to also shift the target's output distribution, which triggers the semantic preservation constraint and partially cancels the attack. Mistletoe resolves this with null-space projection: degradation gradients are projected into the null space of the local semantic-preserving direction, removing their component that would shift visible outputs while preserving their effect on draft acceptance. The two objectives become approximately orthogonal.

This is the part that makes the attack practically relevant. Without null-space projection, you'd face a tradeoff — make the draft disagree more, but accept some visible output degradation. With it, the two objectives decouple enough that the attack can achieve substantial acceptance-length collapse without touching perplexity.

## Why This Matters Now

A few years ago, speculative decoding was an optimization technique used by people with the resources to run and maintain separate draft models. Now it's infrastructure. The attack described in Mistletoe applies to every system that separates the drafting and verification steps — which is nearly every production serving setup that cares about throughput.

The threat model isn't particularly exotic: an adversary who can craft or inject inputs (or who controls a prompt injection in an agentic pipeline) could systematically degrade serving throughput without the operator having a clean signal to detect it. Standard monitoring checks for accuracy, toxicity, and coherence — not for unexplained latency increases that correlate with specific input patterns. The paper notes that perplexity and output quality remain unchanged, which means the normal quality monitors don't catch it.

The defense space is less developed. Rate-limiting obviously doesn't help if requests look legitimate. Monitoring acceptance-length distributions per-input is a possible approach but requires per-request telemetry that most systems don't currently expose. Adaptive rejection of inputs with anomalously low acceptance rates would work, but requires calibrating what "anomalous" means per deployment. None of this is hard to build, but none of it is standard today.

The deeper issue is that speculative decoding's speedup guarantee comes with an implicit assumption: the input distribution at inference time matches the distribution the draft model was calibrated on. Mistletoe shows you can violate that assumption on purpose. The paper is a reminder that the gap between draft and target model distributions isn't just a performance engineering problem — it's an attack surface.
