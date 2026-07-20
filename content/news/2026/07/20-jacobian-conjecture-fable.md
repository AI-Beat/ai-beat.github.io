---
title: "An 87-Year Conjecture Falls During the World Cup"
date: 2026-07-20T06:09:49+00:00
draft: false
slug: jacobian-conjecture-fable-counterexample
categories: [research]
tags: [mathematics, anthropic, claude-fable, formal-proofs]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic researcher and mathematician Levent Alpöge used Claude Fable
    during the World Cup final to produce a concrete, checkable counterexample
    to the Jacobian Conjecture — a problem on Smale's 1998 list of Mathematical
    Problems for the Next Century. The map is three polynomials in three
    variables. The Jacobian determinant is -2. The conjecture is false.
---

Late yesterday, [Levent Alpöge](https://x.com/__alpoge__/status/2079028340955197566) — a mathematician who completed his PhD at Princeton under Fields Medalist Manjul Bhargava, held a Junior Fellowship at Harvard's Society of Fellows, and now works at Anthropic — posted on X that the Jacobian Conjecture is false, with an assist from "my close friend fable for working during the world cup final." The counterexample is compact enough to fit in a tweet.

The [Jacobian Conjecture](https://en.wikipedia.org/wiki/Jacobian_conjecture), posed in its general form in 1939 by Ott-Heinrich Keller, asks whether a polynomial map \(F: \mathbb{C}^n \to \mathbb{C}^n\) whose Jacobian determinant is a nonzero constant must be invertible (with a polynomial inverse). It appears on Stephen Smale's 1998 list of Mathematical Problems for the Next Century. For 87 years it was neither proved nor disproved; partial results accumulated, but no one found either a proof or a concrete counterexample.

The counterexample Alpöge posted is a map \(F: \mathbb{C}^3 \to \mathbb{C}^3\):

\[
F = \bigl((1+xy)^3 z + y^2(1+xy)(4+3xy),\; y + 3x(1+xy)^2 z + 3xy^2(4+3xy),\; 2x - 3x^2 y - x^3 z\bigr)
\]

The Jacobian determinant of this map is \(-2\) — a nonzero constant, satisfying the conjecture's hypothesis. But \(F\) sends three distinct points to the same output:

\[
(0,\, 0,\, -\tfrac{1}{4}),\quad (1,\, -\tfrac{3}{2},\, \tfrac{13}{2}),\quad (-1,\, \tfrac{3}{2},\, \tfrac{13}{2})
\]

all map to \(\bigl(-\tfrac{1}{4}, 0, 0\bigr)\). A function that isn't injective can't have an inverse. The conjecture is false.

What makes this unusual isn't just the mathematical content. The Hacker News thread on this [filled quickly](https://news.ycombinator.com/item?id=48973869) with people verifying the claim in SageMath and SymPy. One early commenter noted the result is "straightforwardly checkable" — and that's the right word. Unlike a long human-written proof that requires domain expertise to audit, this counterexample is elementary to verify once you have it. You can check the Jacobian computation and the three preimage evaluations by hand with patience, or in seconds with a computer algebra system.

That gap — between difficulty of *finding* and difficulty of *checking* — is where AI-assisted mathematics has been most productive this month. Nine days ago, a multi-agent system produced a [proof of the Cycle Double Cover Conjecture](https://ai-beat.github.io/news/2026/07/cycle-double-cover-proof/). This past weekend, Phillip Kerger [closed a 30-year gap](https://ai-beat.github.io/news/2026/07/convex-optimization-gpt56-lower-bound/) in zeroth-order convex optimization using GPT-5.6 Sol Pro over two conversational sessions. The Jacobian Conjecture result is the most compact of the three — just a polynomial triple, the most down-to-earth possible form a mathematical result can take.

The mode here is worth noticing: it wasn't a scaffolded agent harness, it wasn't a 64-model parallel search. It was a mathematician who knew what problem he was working on, using a language model as a thinking partner on a specific hard subproblem, apparently over a couple of hours on a Sunday evening. Alpöge credits a friend for asking about the problem in the first place, which suggests the session may have started as exploration rather than a targeted hunt.

What Claude Fable supplied remains opaque from the outside — the tweet doesn't describe the working session. It might have generated candidate constructions for Alpöge to test, or worked through the algebra in a direction Alpöge suggested, or something else entirely. The nature of the collaboration matters for understanding what actually happened, and that information isn't yet public. There's an obvious footnote here: Alpöge works at Anthropic. Claude Fable is an Anthropic model. The tweet calling it "my close friend fable" is presumably closer to the literal truth than most anthropomorphizations of AI.

What is public is the counterexample, and it checks out. The mathematical community will now want to understand whether the construction generalizes — whether the conjecture is false in other dimensions or for other reasons — and to situate this within the existing theory. But the primary claim is done: the Jacobian Conjecture is settled, and it's settled false.
