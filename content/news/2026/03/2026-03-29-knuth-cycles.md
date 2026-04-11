---
title: "Shock! Shock! — Knuth, Claude, and the Three-Way Mathematical Proof"
date: 2026-03-29T11:00:00+01:00
draft: false
slug: knuth-claude-cycles-proof-assistant
tags: [research, mathematics, reasoning, formal-verification]
categories: [research]
params:
  author: AI Beat Desk
  summary: >-
    Donald Knuth published a paper in early March titled "Claude's Cycles" — named after
    the AI that spent an hour finding an algorithm for a directed graph decomposition
    problem he had been stuck on for weeks. Knuth wrote the formal proof himself; Claude
    did the search. Now a Lean 4 formal verification of the theorem, built with Claude and
    a proof agent toolkit, closes the loop. The three-stage division of labor — AI
    explorer, human prover, machine verifier — is a concrete model worth examining.
---

In early March, Donald Knuth published [a short paper](https://www-cs-faculty.stanford.edu/~knuth/papers/claude-cycles.pdf) with an unusual opening. "Shock! Shock!" The problem he'd been unable to crack for weeks had just been solved by Claude Opus 4.6 in about an hour. He named the paper "Claude's Cycles."

The problem is specific enough to be worth stating. Consider an \(m \times m \times m\) grid of points where each point \((i, j, k)\) has three outgoing arcs: one increments the first coordinate \(\bmod\ m\), one the second, one the third. The graph is a Cayley digraph, every node with out-degree 3. The question: can you partition all arcs into exactly three directed Hamiltonian cycles? Knuth had verified solutions computationally for odd \(m\) up to \(16 \times 16 \times 16\) but couldn't find a general construction. He asked colleague Filip Stappers to try Claude.

What followed was 31 exchanges over roughly an hour, and it did not go smoothly. Claude failed roughly 14 times in a row — trying linear formulas, brute-force search, Gray code variants, simulated annealing — before finding the thing that worked. The pivot came when it reformulated the problem using fiber coordinates, where a single value \(s = (i+j+k) \bmod m\) determines which fiber each point belongs to. From that angle, the construction that worked turned out to correspond to the classical modular m-ary Gray code, a well-known combinatorial structure Claude derived from scratch without recognizing what it was rediscovering.

Stappers tested it for every odd m up to 101. It held every time.

The division of labor at that point was precise. Claude produced an empirical construction that worked in every tested case. It did not — and could not — produce a proof. Knuth then did what a mathematical collaborator does: read the pattern, understood why it worked, and wrote the rigorous argument. Setting up an exact cover problem on the 11,502 Hamiltonian cycles for the \(m=3\) case, he found exactly 4,554 valid decompositions, of which 760 involve only generalizable cycles — the full space that makes the construction provably valid for all odd \(m > 1\).

"It seems I'll have to revise my opinions about generative AI one of these days," Knuth wrote. That sentence is calibrated: he isn't announcing that AI can do mathematics. He's saying his prior was too dismissive, and that what happened is real enough to update on.

The story picked up again today via a Bo Wang [thread on Hacker News](https://news.ycombinator.com/item?id=47557166) about what came next. Mikito Iwamasa, working in mid-March, built [a complete Lean 4 formal verification of the theorem](https://medium.com/@mikito3/how-i-build-a-proof-on-the-prof-donald-knuths-claud-s-cycle-with-opus-4-6-and-lean4-skills-62b99a982f67). Claude Opus 4.6 again, alongside a toolkit called [lean4-skills](https://github.com/cameronfreer/lean4-skills) that provides a "proof agent" for automated repair of failed proof steps. Claude generated theorems and lemmas; when compilation errors appeared, repair agents were dispatched to fix them one by one. The Axiom Lean Engine verified the result.

What this three-stage arc documents is a real and specific division of labor: language model as explorer, human as prover, proof assistant as verifier. Each is handling what the others handle poorly.

Language models are good at high-breadth search. Given a problem with a large candidate space, they can try many approaches quickly, reformulate, spot numerical patterns, and refocus. Claude's 14 failures before the correct construction are a feature of this: it kept searching where a human might have stopped. That persistence isn't intelligence in the traditional sense, but it's useful precisely because the search space was large and the answer wasn't obvious.

What language models can't reliably do is establish that a construction works in general. Claude found the pattern for the cases it tested. It didn't know it was rediscovering Gray code, and it couldn't have written the "if and only if" theorem that pins down exactly which constructions are generalizable. That required Knuth — reading the output, understanding the structure, and producing a proof that closes off all remaining cases.

The proof assistant then removes Knuth's fallibility from the permanent record. A human-authored proof can have gaps that a careful reader skips over. Lean 4 does not skip over gaps. The Iwamasa formalization provides a certificate independent of whether the paper's prose is fully rigorous.

The even-dimensional cases remain open. Claude made no meaningful progress on those, and the odd-m solution's elegance may be specific to the structure that fiber coordinates expose. Whether a similar three-stage process works there is an open question.

What Knuth's "Shock! Shock!" honestly captures is something more limited than "AI does math now" and more interesting than "AI can't do real math." It's a specific capability — sustained guided search with rapid iteration — that doesn't eliminate the need for human mathematical understanding, but does change what the human needs to do first.
