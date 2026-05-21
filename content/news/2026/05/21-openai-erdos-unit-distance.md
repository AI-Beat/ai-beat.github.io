---
title: "Eighty Years, One Model, One New Idea"
date: 2026-05-21T06:08:47+00:00
draft: false
slug: openai-erdos-unit-distance-conjecture
categories: [research]
tags: [math, reasoning, openai, proof-generation]
params:
  author: AI Beat Desk
  summary: >-
    An internal OpenAI reasoning model disproved a conjecture in discrete
    geometry that had been open since 1946. It found a polynomial improvement
    to the best known lower bound for the planar unit distance problem —
    n^(1+δ) with δ = 0.014 — by importing tools from algebraic number theory
    that no human mathematician had previously applied to this problem. The
    proof was verified and endorsed by several leading mathematicians, including
    Fields Medalist Tim Gowers.
---

Paul Erdős posed the planar unit distance problem in 1946. It is one of those deceptively simple questions in combinatorial geometry: place n points anywhere on a flat plane. How many pairs of points can sit at exactly one unit apart? Call that maximum u(n). What does u(n) look like as n grows large?

For 80 years, the best known lower bound came from a construction using Gaussian integers — points at positions a + bi where a and b are integers, scaled to unit spacing. This grid gives roughly \(n^{1 + c / \log \log n}\) unit-distance pairs, which is technically super-linear but barely: the exponent creeps above 1 so slowly that for any practical n the construction is nearly indistinguishable from linear. The upper bound, meanwhile, sits at \(O(n^{4/3})\) — proved by Spencer, Szemerédi, and Trotter in 1984 and not meaningfully improved since. Between these two bounds, an enormous gap sat open for decades, and the Gaussian integer construction was widely understood to be the natural, essentially-optimal approach for the lower end.

An [internal OpenAI reasoning model](https://openai.com/index/model-disproves-discrete-geometry-conjecture/) has now closed part of that gap in a qualitatively different way. The model found constructions showing that for infinitely many values of n, you can arrange n points to achieve at least \(n^{1+\delta}\) unit-distance pairs — with δ a positive constant. Will Sawin at Princeton subsequently refined the bound to pin δ = 0.014. That is a small exponent, but the shift from \(n^{1 + c/\log \log n}\) to \(n^{1.014}\) is not a minor quantitative improvement: it crosses a qualitative threshold. The two expressions have fundamentally different asymptotic behavior. The old bound barely edges past linear; the new one guarantees a genuine polynomial excess.

## Where the Idea Came From

What surprised researchers most was not the conclusion but the method. Human mathematicians had been working within the geometry itself — thinking about grids, lattices, circle arrangements, algebraic curves. The model went sideways into a completely different area of mathematics: algebraic number theory.

The proof imports tools from [class field theory](https://en.wikipedia.org/wiki/Class_field_theory), specifically the existence of infinite class field towers, which was established in the 1960s via [Golod–Shafarevich theory](https://en.wikipedia.org/wiki/Golod%E2%80%93Shafarevich_theorem). The key idea is this: Gaussian integers correspond to the ring of integers in \(\mathbb{Q}(i)\), the simplest imaginary quadratic field. The Gaussian integer grid construction works because points at Gaussian integer positions have controlled unit-distance structure. The model's insight was that you can extend this to points corresponding to integers in a larger algebraic number field — specifically one reached by a carefully chosen tower of field extensions — and that the richer arithmetic structure of those fields allows you to pack more unit-distance pairs into the same number of points.

Infinite class field towers are primarily a tool for studying ramification and factorization in algebraic number theory. The fact that they yield anything useful about geometric point configurations is not obvious. There is no prior literature connecting Golod–Shafarevich to combinatorial geometry; the model appears to have synthesized this link from scratch.

## The Verification

The [technical write-up](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-remarks.pdf) was checked by a group of external mathematicians including Noga Alon, Melanie Wood (Harvard), Thomas Bloom (Oxford), and Will Sawin (Princeton). Bloom is notable here: he publicly criticized OpenAI in October 2025 when a previous model incorrectly claimed to have proved a different mathematical result, so his sign-off carries extra weight. Fields Medalist Tim Gowers said he would recommend the proof for publication without hesitation — and noted this is the first AI-produced mathematics he has seen that he would describe as containing a genuinely original idea rather than a well-structured search through existing technique space.

OpenAI has not disclosed which model produced the result, describing it only as an internal reasoning model.

## What This Does and Doesn't Mean

It is worth being precise about what the result establishes, since the framing of "AI solves math problem" gets applied to quite different things. This result is not: a model regurgitating a known proof from training data, or a brute-force search over a bounded configuration space, or a computer-assisted verification of a human-designed proof. It is: a model independently identifying a non-obvious cross-disciplinary connection, constructing an argument using techniques from a different mathematical domain, and producing a proof that mathematicians with domain expertise have verified as correct and novel.

The gap between u(n) and n^{4/3} remains enormous. Nobody is close to the upper bound. δ = 0.014 is not a dramatic improvement in an absolute sense. But crossing the qualitative threshold — establishing a fixed positive exponent — after 80 years of stagnation on this problem is exactly the kind of thing that moves the needle in discrete geometry.

What the result most directly demonstrates is not that AI will replace mathematicians, but that reasoning models can now produce cross-disciplinary insights that human specialists, working within their domains, hadn't generated despite decades of effort. Whether this was a lucky synthesis or represents a repeatable capability is still an open question. The field now has one clean proof that the answer is not "never."
