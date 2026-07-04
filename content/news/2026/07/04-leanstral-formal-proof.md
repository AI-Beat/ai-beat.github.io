---
title: "miniF2F Hits the Ceiling"
date: 2026-07-04T06:00:00+00:00
draft: false
slug: leanstral-formal-proof-saturation
categories: [research]
tags: [formal-methods, theorem-proving, mistral, lean4, open-source, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Mistral's Leanstral 1.5 scores 100% on miniF2F and solves 587 of 672
    Putnam Competition problems using a 6B-active-parameter MoE. The model
    saturates the main formal-proof benchmark and finds real bugs in production
    code — at roughly $4 per Putnam problem versus competitors charging $300.
---

When a benchmark maxes out, the field has to find harder questions. [Leanstral 1.5](https://mistral.ai/news/leanstral-1-5/), released by Mistral on June 30, just did that to miniF2F: 100% on both the validation and test sets. That benchmark — 488 problems from AMC, AIME, and IMO competition mathematics, formalized in Lean and Isabelle — has been the main leaderboard for automated theorem proving for years. It is now saturated.

The architecture is unusual for what it accomplishes. Leanstral 1.5 is a 119B mixture-of-experts model with only 6B active parameters per forward pass. Most of the network sits idle on any given token. The model was trained in three stages: mid-training on domain-specific proof corpora, supervised fine-tuning, and then reinforcement learning using CISPO, a variant of online RL designed for structured generation tasks. Context length is 256k tokens — long enough to hold a full proof file alongside its dependencies.

The harder benchmarks are more informative. PutnamBench pulls from the William Lowell Putnam Mathematical Competition, the most prestigious undergraduate math contest in North America. Problems are famously open-ended: you're expected to find the right framing, not just execute a procedure. Leanstral 1.5 solves 587 of 672, which puts it well above the previous best. On [FATE](https://arxiv.org/abs/2511.02872), a newer series of abstract algebra benchmarks that deliberately targets the gap between contest math and graduate-level work, the model hits 87% on FATE-H (graduate-level) and 34% on FATE-X (PhD qualifying exam difficulty). FATE-X was essentially unsolved by previous models; the previous state of the art on FATE-H was in the single digits.

What makes this worth taking seriously beyond benchmarks is the code verification work. The blog post describes deploying the model on real codebases and finding five previously unknown bugs — including an integer overflow in a zigzag decoding function. That is the actual use case formal methods researchers have been chasing for thirty years: not just proving abstract math, but verifying that software does what it claims to do. The gap between competition math and production code is enormous, and five bugs in real repositories is more evidence of practical utility than a lot of benchmark tables.

Test-time compute scaling is linear and predictable. Performance improves monotonically as token budget increases from 50k to 4M tokens, with no apparent plateau in the tested range. This matters because it means you can dial the cost-quality tradeoff to match your requirements. On PutnamBench, Mistral estimates the model costs roughly $4 per problem solved. The next best system, [Seed-Prover 1.5](https://arxiv.org/pdf/2512.17260), costs an estimated $300 or more per problem. That is a 75x cost reduction for comparable or better results.

The model is Apache 2.0, weights available on Hugging Face, and free through Mistral's Labs tier API. That combination — genuinely strong on hard problems, cheap to run, openly licensed — is not what formal verification tooling looked like two years ago. The field has been making tools that researchers at well-funded universities could use if they were sufficiently patient. This is something a working engineer can call from their CI pipeline.

The caveat is that formal verification requires writing code in Lean 4, which has a steep learning curve and a relatively small but growing ecosystem. The model can find bugs and generate proofs, but you still have to express your software's correctness properties as Lean specifications for any of it to be useful. That adoption barrier is real and probably determines how quickly these capabilities translate from impressive benchmark numbers into production tooling. The tooling for writing Lean specifications at scale is improving but isn't yet at the point where you'd add it to an arbitrary codebase without significant up-front investment.

Still: a model that can solve 87% of graduate-level abstract algebra problems, costs $4 per Putnam solution, runs on open weights, and finds bugs in real code is qualitatively different from what existed before. The benchmark saturation is mostly a call to build harder benchmarks. The code verification results are the more interesting number.
