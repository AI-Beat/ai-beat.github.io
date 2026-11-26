---
title: "The Last Assumption Falls"
date: 2026-11-26T06:11:46+00:00
draft: false
slug: cogentic-factoring-discrete-log
categories: [research]
tags: [cryptography, complexity-theory, factoring, cogentic, math, google]
params:
  author: AI Beat Desk
  summary: >-
    Google's Cogentic system dropped six math preprints on November 25, including
    one claiming unconditional L[1/3, O(1)] integer factoring — removing the
    heuristic assumptions that have propped up the number field sieve's complexity
    claim for thirty years. The practical threat to RSA is essentially nil (the
    constant is 105, not 1.92), but the theoretical gap is now closed.
---

For thirty years, the best-known algorithm for integer factoring — the general number field sieve — has had a known running time of \(L_N[1/3, (64/9)^{1/3}]\), which works out to roughly \(L_N[1/3, 1.92]\) in the standard L-notation for subexponential algorithms. The catch is that this analysis rests on heuristic assumptions that have never been proven. The sieve works in practice, and the heuristics are widely believed, but there has been no unconditional proof that factoring is in \(L[1/3]\). That's been an irritating gap in the theory for a generation.

On November 25, [Google Research's GitHub](https://github.com/google-research/google-research/tree/master/cogentic_research_papers) published six math preprints attributed to [Cogentic](https://cogentic.github.io/), their multi-agent research system built on Gemini 4. Three of them bear directly on factoring and discrete logs. The most significant is titled *"Unconditional Integer Factorization in Time L_N[1/3, O(1)]"* — which is exactly what it sounds like: a claimed proof that factoring an N-bit integer sits in \(L_N[1/3, O(1)]\) with no unproven assumptions. A companion paper claims a *deterministic* subexponential factoring algorithm in \(L_N[3/4]\) time, and the third claims sub-quasi-polynomial discrete logarithms for finite fields of small fixed characteristic.

The obvious question is whether this threatens RSA. The answer is no, and the reason is the constant. The unconditional factoring paper achieves \(L[1/3, 105]\), not \(L[1/3, 1.92]\). Because L-notation buries the constant inside an exponential — \(L_N[1/3, c] = \exp(c \cdot (\ln N)^{1/3} (\ln \ln N)^{2/3})\) — a constant of 105 versus 1.92 is not a small difference. At 2048-bit keys, the gap in actual operations is immense. No one is factoring RSA-2048 with this. The deterministic \(L[3/4]\) algorithm is slower still. The discrete-log result targets small-characteristic fields (think fields built over characteristic 2 or 3), which the cryptographic community largely abandoned after Barbulescu, Gaudry, Joux, and Thomé's [2014 quasi-polynomial attack](https://arxiv.org/pdf/1306.4244.pdf) anyway.

What actually closes is the theoretical gap. After three decades of heuristics propping up the asymptotic claim, there's now a proof — if the Cogentic preprints check out. The papers come with no Lean verification (unlike the [Riemann Hypothesis proof](https://github.com/lsernau/grh-lean) that made the rounds last week), so they'll need traditional peer review before the community accepts them. No expert assessment had appeared publicly as of this writing.

The broader context is worth noticing. This is six research papers in a single day, all from a system Google describes as operating largely autonomously. Cogentic's output isn't confined to the factoring cluster: the same batch includes results on Nash equilibria, prophet inequalities, the log-rank conjecture, and graph reconstruction — a spread across combinatorics, complexity, and game theory. The pace of machine-generated mathematics has become difficult to absorb even for people whose job it is to keep up.

Scott Aaronson's [reaction post yesterday](https://scottaaronson.blog/?p=10379) captures the mood well. He calls the OpenAI matrix multiplication lower bound "another extraordinary result" and says he now feels "in his bones" that a proof of P ≠ NP isn't far away — a claim that would have sounded unhinged twelve months ago. He's not claiming the proofs are definitely correct. His point is more that the credentialing mechanism has changed: when a lab ships results with machine-checkable Lean certificates, and those certificates have been verified, disproving them takes more than intuition.

The Cogentic papers don't come with Lean proofs. So for now they sit in the same pile as any ambitious preprint — interesting claims that require scrutiny. The factoring one in particular would be historically significant if it holds. Getting from heuristic to unconditional for \(L[1/3]\) has been an open problem since Pomerance framed it in the 1980s. Whether Cogentic cracked it, or produced a subtle error at scale, is something number theorists will spend the next few months figuring out.
