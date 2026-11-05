---
title: "Prove It: Anthropic's zkSNARK System for Model Verification"
date: 2026-11-05T06:13:25+00:00
draft: false
slug: flashprove-provable-inference
categories: [research]
tags: [cryptography, zero-knowledge, inference, trust, anthropic]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic published FlashProve on November 2 — an open-source prototype that uses
    zkSNARKs to let users verify which model weights actually produced a given response.
    At ~110 tokens/sec proof generation on a single H100 (with 70–95x GPU overhead),
    it's not production-ready, but the problem it's solving is real and underexplored.
---

Here's a problem that's been quietly growing: when you send a prompt to an API, you have no way to verify which model actually responded. The provider says it's Claude 4 Sonnet or GPT-6; you have to take their word for it. As inference costs drop and competitive pressure rises, model substitution — serving a cheaper or older model while billing for a newer one — becomes an increasingly plausible failure mode. And even without bad faith, misrouting happens.

[Anthropic's FlashProve](https://www.anthropic.com/research/provable-inference), published November 2 by a team that includes collaborators from EPFL, MIT, StarkWare, and Succinct, is an open-source prototype that addresses this directly using zero-knowledge succinct arguments (zkSNARKs).

## What it does

The core idea: a provider commits to a set of model weights in advance. When generating a response, they also generate a zero-knowledge proof that the response came from those committed weights. A user (or auditor) can verify this proof without ever seeing the weights.

The practical numbers: on Llama 4 Maverick (a 400B MoE model at 4-bit quantization), one H100 generates proofs at about **110 tokens/second**, with proofs of roughly **19 MB per 1,024-token request**. The CPU verifier checks a proof in about **17 seconds** on 24 threads. The one-time offline commitment to the weights takes about 39 minutes.

The overhead is significant: proving takes roughly **70 to 95 times the GPU time** of running the same prompts through vLLM. But the team notes this is far faster than prior open-source systems — the previous best open-source zkSNARK inference systems were much slower. The design runs on a single GPU by streaming weights from host memory, which keeps hardware requirements bounded even for large models.

## Why zkSNARKs work here

The standard problem with zero-knowledge proofs is that they're expensive. Neural network inference involves billions of floating-point operations, and proving each one is costly. FlashProve sidesteps part of this by working with an **integer version of the weights** under **greedy decoding**. This is a meaningful constraint: sampled output (temperature > 0) is not covered, and floating-point arithmetic is not covered either. What you get is a proof that the greedy response came from a specific quantized version of a model — which is useful for auditing purposes even if it doesn't cover every inference scenario.

The completeness, soundness, and zero-knowledge properties are stated and proven in Lean. The fast C++ verifier doesn't carry the formal guarantee, which is an honest disclosure. The Lean spec, paper, and fast implementation are described as not guaranteed to match each other — also honest.

## Why this matters

The problem FlashProve addresses isn't hypothetical. In regulated industries — healthcare, finance, legal — there's often a compliance requirement that a specific vetted model was used for a specific decision. Right now, the only way to enforce this is contractual, with periodic audits that check spot samples. Cryptographic proofs would let you audit every response rather than a sample.

More speculatively: as AI agents operate with increasing autonomy and handle consequential decisions, knowing which model (and version) made a decision becomes important for liability, debugging, and governance. The EU AI Act has provisions that point in this direction.

The 70–95x overhead puts this firmly in the "not ready for real-time production inference" category. But the trajectory matters: prior systems were even slower, and zkSNARK hardware acceleration (specialized ASICs, GPU kernels, FPGA implementations) is an active area. FlashProve's Apache 2.0 open-source release gives the broader community something concrete to build on.

The interesting question isn't whether the overhead will come down — it almost certainly will — but whether the market will develop a demand for verified inference before or after the technology becomes cheap enough to run everywhere.
