---
title: "Thirty Years of Queries"
date: 2026-07-19T06:09:53+00:00
draft: false
slug: convex-optimization-gpt56-lower-bound
categories: [research]
tags: [mathematics, openai, formal-proofs, optimization]
params:
  author: AI Beat Desk
  summary: >-
    A UC Berkeley IEOR researcher used GPT-5.6 Sol Pro over two chat sessions
    totaling roughly four hours to prove a lower bound in zeroth-order convex
    optimization that had resisted attempts for 30 years, then formalized the
    result in Lean 4. A different kind of AI-does-math story than the CDC proof:
    one expert, one model, one hard problem.
---

A week after OpenAI's [Cycle Double Cover Conjecture](https://ai-beat.github.io/news/2026/07/cycle-double-cover-proof/) post generated waves in the math community, a quieter but structurally distinct data point arrived on Reddit, and then Hacker News. Phillip Kerger, who teaches industrial engineering and operations research at UC Berkeley, [posted](https://news.ycombinator.com/item?id=48957779) that he had used GPT-5.6 Sol Pro to close a 30-year gap in a problem he'd been working on sporadically for about a year.

The problem is in deterministic zeroth-order convex optimization: given a convex, 1-Lipschitz function \(f : B_d \to \mathbb{R}\) defined on the Euclidean unit ball in \(\mathbb{R}^d\), and an oracle that returns exact real values \(f(x)\) at any queried point, how many queries does a deterministic algorithm need to find an approximate minimizer? The upper side of this was understood: a 30-year-old algorithm achieves the task in \(O(d^2)\) evaluations (up to accuracy-dependent factors). The lower side was open. The question was whether \(O(d^2)\) is actually tight, or whether something cleverer could do better. After 30 years, no matching lower bound existed.

Kerger had previously tried GPT-5.4 and GPT-5.5 on the same problem, specifically on a construction approach he believed could work. Both models got far enough to be interesting before losing the thread. GPT-5.6 didn't lose it. Two sessions — the first running 148 minutes, the second 230 minutes — produced a complete proof of an \(\Omega(d^2)\) lower bound, establishing that the old algorithm is essentially optimal in the dimension parameter.

The [GitHub repository](https://github.com/PhillipKerger/zero-order-bounds-lean-verification), titled "Closing the Oracle-Complexity Gap in Derivative-Free Convex Optimization," contains the paper and full Lean 4 formalization, including the spherical-averaging and Brunn-Minkowski arguments that the proof relies on. That last part matters: this isn't a case where the LLM produced plausible-looking prose that might or might not be a proof. The Lean verification means the logical structure has been mechanically checked.

## What makes this different from the CDC result

The OpenAI CDC proof used 64 subagents running in parallel for under an hour, with a long, carefully engineered prompt telling the system to search broadly and avoid premature termination. It was an impressive infrastructure demonstration — a company's formal harness applied to a hard conjecture.

Kerger's result is the opposite shape. One person. One model. Two conversational sessions. No multi-agent scaffold. The prompt engineering was substantive but came from domain expertise: Kerger already knew the construction direction that was likely to work, and could recognize when the model's reasoning was heading somewhere productive versus drifting. He guided it. Without that guidance — without someone who'd thought about the problem hard enough to know where the difficulty lives — it's not obvious the model would have closed the gap rather than producing something locally coherent but globally broken.

This is the mode that seems most plausible as a near-term pattern for AI-assisted mathematical research: a domain expert with a specific open problem using a frontier model as a reasoning partner to explore a construction they already suspected but couldn't complete alone. The model provides raw computational legibility — it can hold a long and intricate argument in context and generate the next steps consistently — while the human provides taste, direction, and the ability to recognize when something is promising versus wrong.

The model also seems to have gotten genuinely better at this. GPT-5.5 couldn't complete the construction that 5.6 did, on the same problem, with the same approach. That's a meaningful signal rather than noise: the gap wasn't in what the human was trying to do, it was in whether the model could carry out the mathematical work required to do it.

The claim from Kerger in the HN thread: the result will appear as a paper, the Lean code is already public. Whether the proof survives scrutiny by other optimization researchers is the next question — but with formal verification already in place, the bar for "survives scrutiny" is lower than for most human-written proofs of similar ambition.
