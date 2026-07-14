---
title: "A Language Designed for Code That Writes Itself"
date: 2026-07-14T06:00:00+00:00
draft: false
slug: jacquard-lang-ai-written-code
categories: [tooling]
tags: [programming-languages, effects, ai-tooling, code-review]
params:
  author: AI Beat Desk
  summary: >-
    Jacquard is a research programming language that puts effects, uncertainty,
    and content-addressed identity directly in the syntax — on the premise that
    if machines write most code, human reviewers need the language itself to
    answer "what can this touch, and how sure are we."
---

Every programming language ever designed has made the same implicit assumption: a person wrote this code. Humans understand context, can hold global state in their heads, and will catch problems by reasoning about what they intended. The language can afford to be terse about side effects, hand-wavy about uncertainty, and loose about identity across refactors — because the author can fill in the gaps.

[Jacquard](https://github.com/jbwinters/jacquard-lang) is a small research language that throws out that assumption. Its premise: when most code is written by a machine-learning model and reviewed by a person, the reviewer needs the language itself to answer two questions fast — *what can this code touch?* and *how confident should I be in its behavior?* The author isn't around to explain.

The language picks three places where existing languages stay silent and makes them explicit.

**Effects in signatures.** A function like `(text) ->{net} text` tells you before you read the body that it may reach out to a network. The runtime enforces this: if you invoke code that touches the filesystem without granting `--allow fs`, it fails. This isn't new — [Koka](https://koka-lang.github.io/) and [Frank](https://github.com/frank-lang/frank) have algebraic effects, and Rust has `unsafe` for a narrower version of the same idea. What's different here is the motivation. In Jacquard, effect annotations exist specifically so reviewers can triage AI-generated code by blast radius before reading it. The effect system is the diff summary.

**Uncertainty as library code.** Jacquard has no special probabilistic syntax. Instead, it ships algebraic effect handlers for exact Bayesian inference over finite discrete models — sample a weighted choice, record evidence, enumerate exact probabilities. The multi-shot handlers let you run the same code against different continuations, which means probabilistic inference falls out without a separate runtime or framework. Whether this matters in practice depends on whether AI-generated code commonly expresses uncertainty (currently, mostly no — LLMs tend to commit to deterministic paths), but it's interesting as a design position: *this is where non-determinism lives in the language, not in comments or the model's system prompt.*

**Content-addressed identity.** Jacquard hashes programs on canonical resolved structure — the underlying semantics — rather than source formatting. Renaming a variable, reformatting a block, or adding a comment doesn't change a program's identity. This means a pure function re-run check can skip re-execution when nothing semantically changed. It also means identity is stable across AI-generated style variations; if two models generate the same logic with different variable names, they produce the same hash.

None of this is ready to ship. Jacquard 0.1-rc3 is explicitly a research prototype — the README flags limitations, the test suite is sparse, and there's no ecosystem. But prototypes don't need ecosystems to be useful. This one is useful for the questions it asks clearly.

The interesting tension here: the three features Jacquard surfaces — effects, uncertainty, identity — are also the three things that make AI-generated code hardest to audit. Models reach for network and filesystem access freely, express false confidence in their outputs, and are indifferent to naming and formatting. A language that makes all three visible in the syntax doesn't solve the problem, but it concentrates the problem into the parts of the code a human reviewer would most want to inspect.

Whether any production language will adopt this framing is genuinely unclear. Adding an effect system to an existing language is expensive (Swift tried and shelved it; Rust has spent years on async effects). The path most likely to matter is tooling rather than language — linters and static analyzers that approximate effect inference post-hoc, or sandboxed runtimes that track capability grants at the function level. But Jacquard's value is as a proof of concept for what it would look like if we designed the language from scratch with AI authorship in mind. That framing will only become more relevant.
