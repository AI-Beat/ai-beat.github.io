---
title: "Palomar: A Notary for AI-Generated Proofs"
date: 2026-08-19T06:10:00+00:00
draft: false
slug: palomar-lean-math-registry
categories: [research]
tags: [math, lean, formal-verification, ai-generated-proofs]
params:
  author: AI Beat Desk
  summary: >-
    Terry Tao and the Lean FRO launched Palomar on August 18 — a registry that
    runs submitted Lean proof repositories through a verification pipeline and
    publishes the exact formal statement, library dependencies, and reviewer
    comments. The project is a direct response to the surge in AI-generated
    Lean proofs, where the bottleneck has shifted from producing proofs to
    trusting them.
---

Last week, Terence Tao [announced Palomar](https://terrytao.wordpress.com/2026/08/18/palomar-a-registry-of-lean-verified-mathematics/) on his blog: a new registry of Lean-verified mathematics, incubated by the Lean FRO and the International Center for AI and Research in Mathematics (ICARM). The project is simple in concept and timely in context.

AI systems are now producing Lean-formalized proofs at meaningful volume. Whether the source is OpenAI Astra, DeepSeek's reasoning models, or researchers using AI-assisted formalization tools, the result is a growing accumulation of Lean repositories claiming to prove things. The problem is that very few people can evaluate whether a given Lean repository actually proves what it claims.

Tao identifies three distinct failure modes. First, the proof might not typecheck at all — the Lean file looks impressive but errors out under verification. Second, the proof might typecheck but rely on shortcuts: `sorry` placeholders (Lean's equivalent of marking a step as assumed), additional axioms beyond Lean's standard foundations, or other holes that let invalid reasoning pass. Third, even a clean proof might formalize a statement subtly weaker or different than the informal claim it purports to verify — matching the Lean type signature to the human-language theorem is a skill of its own.

Palomar addresses all three: it runs submitted proof repositories through a verification pipeline, checks for cheats, and then publishes the exact formal Lean statement alongside which libraries it depends on and the reviewer's comments. The result is a citable, machine-checkable record that non-Lean experts can reference without understanding the proof internals themselves. As a test case, Tao submitted his own recent Lean formalization of the proof of [Sendov's conjecture](https://en.wikipedia.org/wiki/Sendov%27s_conjecture) — a longstanding problem he resolved in 2020 — and it passed.

This is infrastructure for a specific new problem that AI creates rather than solves. Producing a Lean proof used to be slow enough that community trust developed organically: a formalization took months of expert effort, it was read carefully, and its authors were known. Now that AI tools can generate Lean proofs in hours or days, that organic trust mechanism breaks down. Palomar is an attempt to replace it with an explicit, automated audit trail.

The framing that stands out in Tao's post is that Palomar accepts submissions "whether human-generated, AI-generated, or some mixture of both." The registry deliberately doesn't care about provenance — only about correctness. That's a reasonable stance: a proof is valid or it isn't, regardless of how it was produced. But it also means Palomar becomes the natural destination for AI-generated proofs that want credibility, which is exactly the pressure that created the need for the registry in the first place.

Practically, the registry publishes JSON and RSS feeds, organizes results by arXiv identifiers and MSC codes, and is designed to be citable from papers and blog posts. It's modest in scope and honest about what it does: it verifies, it records, it publishes. The hard mathematical work still happens elsewhere.

The interesting longer-term question is whether a registry like this shifts the informal norms around AI-assisted proof publication — not just what gets submitted, but what gets treated as established. A Palomar-verified result carries a different kind of authority than a preprint with a Lean file in the supplementary materials. Whether that difference matters in practice depends on how quickly the broader math community starts citing it.
