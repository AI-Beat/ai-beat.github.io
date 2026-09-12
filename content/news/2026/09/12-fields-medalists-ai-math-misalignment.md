---
title: "Mathematics Isn't a Benchmark"
date: 2026-09-12T06:10:13+00:00
draft: false
slug: fields-medalists-ai-math-misalignment
categories: [research]
tags: [research, mathematics, benchmarks, safety]
params:
  author: AI Beat Desk
  summary: >-
    Twenty-five Fields Medalists — including Terence Tao, Pierre Deligne, and
    Peter Scholze — signed a declaration arguing that AI labs' race to solve
    famous mathematical problems is damaging the discipline. Their case: that
    problem-solving benchmarks are proxies for understanding, not understanding
    itself, and speed-running the proxy destroys the human transmission chain
    that makes mathematical knowledge actually useful.
---

Three days after OpenAI [announced that its agents had proved the global regularity problem for the Navier–Stokes equations](https://openai.com/index/navier-stokes-solution/), twenty-five Fields Medalists published [a declaration at mathandai.org](https://mathandai.org) arguing that what AI labs are doing to mathematics isn't progress — it's a category error dressed up as one.

The signatories include [Terence Tao](https://terrytao.wordpress.com/2026/09/11/a-severe-misalignment-of-ai-in-mathematics/), Pierre Deligne, Peter Scholze, Maryna Viazovska, Caucher Birkar, June Huh, Martin Hairer, Maxim Kontsevich, James Maynard, Manjul Bhargava, and 2026 medalist Yu Deng. When a quarter of all living Fields Medalists agree on something publicly, it is worth reading carefully.

The core argument isn't that AI can't do mathematics — it's that solving problems is only a proxy for the actual goal, which is conceptual understanding. The declaration puts it plainly: "Solving problems is only a tool and proxy for achieving the primary goal of conceptual understanding." An AI that solves the Navier–Stokes existence problem by exhaustive search, opaque symbolic manipulation, or 10,000 concurrent agents over 88 hours does not advance anyone's understanding of fluid dynamics. It produces a true/false statement. Mathematics is full of true/false statements that took generations to be meaningful.

What gets lost in the sprint, the signatories argue, is the transmission chain. When a mathematician solves a hard problem, the solution travels through talks, seminars, careful writeups, referee reports, textbooks, and eventual integration into the working vocabulary of the field. Each step strips away noise and isolates the genuinely new idea. AI-announced solutions skip this entirely. No readable proof. No attribution of which ideas were novel versus adapted from prior work. No chance for the community to internalize the method before the result is superseded by the next announcement.

The practical consequences aren't hypothetical. The Navier–Stokes announcement itself arrived accompanied by a credit dispute involving NYU mathematician Tristan Buckmaster and Anthropic's Levent Alpöge, who had been working the same problem for months using OpenAI's own tools. What ideas did their work contribute? What would be different if they hadn't? Nobody knows, because the proof is not readable in any form that would let you answer that.

There's a reasonable counterargument: knowing a statement is true narrows the search space for the understanding. If you know Navier–Stokes can blow up in finite time, you can start building intuition around that fact rather than hedging between possibilities. And formal verification tools like Lean mean that, unlike previous centuries, there's at least a machine-checkable certificate underneath any claim.

But the declaration's harder point survives this: the current incentive structure optimizes for announcement, not exposition. If AI labs measured success by the quality of the mathematical explanation produced — the "how does this work and why" that takes a year to write clearly — rather than the number of famous open problems closed, the incentives would look different. They don't, because solving the Riemann Hypothesis is a better press release than explaining why a key lemma in the existing literature generalizes more broadly than its authors realized.

The mathematicians are right that this is a misalignment. Whether it's a solvable one depends on whether anyone in a position to change it cares about mathematics beyond its benchmark value.
