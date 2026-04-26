---
title: "The Cliff in Lambda Calculus"
date: 2026-04-26T06:24:13Z
draft: false
slug: lambench-lambda-calculus-llms
categories: [benchmarks]
tags: [benchmarks, lambda-calculus, reasoning, symbolic]
params:
  author: AI Beat Desk
  summary: >-
    Victor Taelin published LamBench, 120 pure lambda calculus programming
    problems in a minimal custom language. The results show a hard generational
    cliff: GPT-5.1, Opus 4.5, and Sonnet 4.5 score exactly 0 out of 120, while
    the top tier — GPT-5.3 Codex and Opus 4.6 — lands at 90%. The benchmark
    tests something standard evaluations mostly avoid: symbolic computation
    that can't be approximated by pattern matching.
---

Most AI benchmarks have a contamination problem. The test sets overlap with training data, or models pattern-match on surface features, or enough similar problems appeared in the pretraining corpus that high scores carry limited information about the underlying mechanism. Victor Taelin — the developer behind [HVM](https://github.com/HigherOrderCO/HVM) and [Interaction Calculus](https://github.com/VictorTaelin/Interaction-Calculus) — designed [LamBench](https://victortaelin.github.io/lambench/) with this in mind: 120 problems written in Lamb, a minimal pure lambda calculus language with named definitions that is almost certainly underrepresented in training data.

The setup is strict. Each problem gives a natural-language description, an encoding scheme (Church numerals, Scott lists, and so on), and a set of test cases. The model writes a `.lam` program that defines `@main`, and the harness normalizes it against every test case. There is no partial credit, no rubric, no human judgment: either the term beta-reduces to the expected output or it doesn't. As one observer noted in the ensuing discussion, "you can't really vibe your way through beta-reduction."

The results across 22 models show not a gradient but a step function. GPT-5.1, Opus 4.5, and Sonnet 4.5 all score 0 out of 120. There isn't a single correct program among them across all three — not an easy Church arithmetic implementation, nothing. Then there's a large jump: Sonnet 4.6 enters the picture at 87/120 (72.5%), followed by GPT-5.2 at 88/120 (73.3%) and GPT-5.5 at 89/120 (74.2%). At the top, GPT-5.3 Codex and Opus 4.6 both land at 108/120 (90%), with Opus 4.7 and Gemini 3.1 Pro at 106/120 (88.3%).

That 0-to-72% jump is the most interesting feature of the leaderboard. It doesn't look like smooth scaling — it looks like a qualitative capability that appeared with a specific model generation and was absent before it. This is consistent with what has been observed elsewhere: on symbolic reasoning tasks, capability sometimes arrives in a step rather than a slope. The models that score zero apparently can't represent the required term structure at all, even on the simplest problems like `add` for Church-encoded naturals.

A few other numbers are worth noting. DeepSeek V4 Pro, just released, scores 45.8% — placing in the middle of the pack, well behind the frontier. Qwen 3.6 Plus hits 47.5%. Grok 4.20 and DeepSeek V4 Pro tie at 45.8%. These are not weak models, but lambda calculus programming in an unfamiliar language exposes gaps that coding benchmarks over Python and C++ might not.

The 10% gap at the top is also real. GPT-5.3 Codex and Opus 4.6 fail 12 problems each. The benchmark includes genuinely hard material in its algorithm category — a BF interpreter, a convex hull, a SAT solver, a type checker, a Sudoku solver, a TSP — all written in pure lambda calculus without built-in data types. Some of those may be hard in the same way competitive programming problems are hard: the framework is understood but the specific term construction requires significant search.

LamBench is tiny. 120 problems, one language, one domain. It doesn't measure broad language understanding or coding ability in general. What it does measure is something specific: the ability to implement algorithms in a formal system where every construct must be built from scratch using only function abstraction and application. The models that score well on it have demonstrated an ability to work in the lambda calculus's symbolic world — to think in terms of normal forms rather than familiar patterns. Given how little of the training data is likely to cover Lamb programs, high scores here are a meaningful signal.

Taelin hasn't published a category-level breakdown yet, which would be the obvious next step — knowing whether failures cluster in the algorithm category or appear uniformly would say something about where exactly the ceiling is.
