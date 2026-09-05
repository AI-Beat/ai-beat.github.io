---
title: "Thirteen Million Lines"
date: 2026-09-05T06:10:00Z
draft: false
slug: flt-lean-formalization
categories: [research]
tags: [formal-verification, math, lean, agents, Claude]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic released a complete, machine-verified formalization of Fermat's Last Theorem in Lean 4. Claude worked for 11 days and produced 13 million lines of proof code—five times the size of Mathlib—checked independently by three separate verifiers. The interesting story is less "AI does math" and more how multi-agent coordination over a theorem dependency graph made it tractable.
---

Fermat's Last Theorem—that no positive integers \(a\), \(b\), \(c\) satisfy \(a^n + b^n = c^n\) for \(n \geq 3\)—sat as an open problem for 358 years before Andrew Wiles proved it in 1995. The proof consumed over 100 pages and took the mathematical community several years to fully check. On September 4, Anthropic [released a formalization](https://www.anthropic.com/research/formalizing-fermats-last-theorem) of that proof in Lean 4: 13 million lines of code, 29,500 intermediate theorems, all verified by three independent methods. Claude produced it in 11 days working largely autonomously.

The achievement is real, but the interesting part isn't the headline number. It's the engineering of how you even attempt something like this.

## The structure of the problem

Formal proof is different from informal mathematics in a way that makes scale extremely painful. In informal math, you can write "by a standard argument" or "it follows that," and a reader who knows the field will fill in the details. Lean doesn't accept that. Every single step—every application of a lemma, every type coercion, every boundary case—must be spelled out in a syntax the kernel can check mechanically. Wiles's proof, translated faithfully, requires an enormous amount of foundational machinery just to state the objects involved, let alone prove properties about them.

The Claude team didn't start from scratch. They built on [Mathlib 4](https://leanprover-community.github.io/), the community's shared mathematics library, plus an [Imperial College London FLT project](https://leanflt.github.io/) that had already formalized substantial pieces of the Galois representation and deformation theory required for the Wiles approach. The agents inherited working implementations of the Frey package and components of Taylor-Wiles, then had to fill in the remaining architecture—which was still enormous.

The key tool was [Prove2Me](https://prove2me.com/), a platform designed around a directed acyclic graph of theorem statements. Every theorem is a node with explicit dependencies. Agents could pick up a leaf node—something with all its prerequisites already proved—attempt a proof, report failure or success, and move on. The DAG structure is what made parallelism tractable: at any point there are many open leaves, and multiple agents can work simultaneously without blocking each other. Failed proofs get re-queued; the system doesn't grind to a halt when one approach doesn't work.

## Scale and cleanliness

The finished proof is [publicly available on GitHub](https://github.com/anthropics/fermats-last-theorem). At 13 million lines it's more than five times the size of Mathlib itself, which is the primary reference library for Lean work done by humans. The code is machine-generated throughout—expect opaque auto-named identifiers and no explanatory comments except citations—but the important metric is whether it's *correct*, not whether it's readable. On that front: the repository has a strict no-sorry policy. In Lean, `sorry` is a way to stub out a hole in a proof and assert it as an axiom. Allowing sorry means your proof has unverified gaps. This formalization uses none. No `native_decide`, no `unsafe`, no `partial def`. The entire thing is axiom-transparent down to Lean's three foundations: propext, Classical.choice, and Quot.sound.

Verification came from three directions. The Lean kernel itself ran a full `lake build`. [leanprover/comparator](https://github.com/leanprover/comparator) cross-checked the final theorem statement against Mathlib's canonical version and returned "Your solution is okay." And [nanoda](https://github.com/ammkrn/nanoda_lib), an independent Lean kernel written in Rust, checked all 1,052,234 declarations in about 30 minutes at 16 threads and found no errors. Getting independent kernel agreement is the gold standard for a claimed proof in formal verification; you're not trusting any single implementation's bug surface.

The resource cost is sobering: roughly six billion output tokens from an internal model comparable to Fable 5.1. Building from source takes 5.5 hours at 96 parallel jobs and peaks around 153 GB of RAM. This was not a compute-light experiment.

## What this actually tells us

There's a version of this story that's mostly hype: AI proves famous theorem, further evidence of superintelligence approaching. That framing misses what's actually demonstrated.

Wiles's theorem was already proved—in 1995, by a human mathematician, rigorously. What didn't exist was a *machine-checkable* version. Formalization is a different task from proof discovery: you're translating human reasoning into a language a computer can verify, which requires precision at a different scale than mathematical intuition requires. The value is that you end up with something you can *trust mechanically*—no review board needed, no possibility of a subtle error that took three years for the community to notice.

What Claude showed is that multi-agent systems can coordinate over very large structured proof graphs and make meaningful progress on tasks where the total scope would be intractable for any individual instance. The DAG architecture—where the platform manages dependencies and agents take individual leaves—is the key insight. It's similar to how coding agents handle large codebases by breaking work into file-level chunks with explicit import dependencies, except the graph structure in formal proof is mathematically exact rather than heuristic.

The natural question is what comes next. The [Riemann Hypothesis](https://en.wikipedia.org/wiki/Riemann_hypothesis) remains open—that would require discovering a new proof, not just formalizing a known one, which is a qualitatively harder problem. But there are plenty of theorems with human proofs that haven't been formalized yet. If the Prove2Me architecture generalizes, reducing the cost of formalization to something a research team can sustain rather than a decade-long community effort, that changes how mathematicians think about what needs checking.
