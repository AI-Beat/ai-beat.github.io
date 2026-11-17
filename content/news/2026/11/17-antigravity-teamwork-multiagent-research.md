---
title: "Automated Exploration With a Truth-Checker Next Door"
date: 2026-11-17T06:16:00+00:00
draft: false
slug: antigravity-teamwork-multiagent-research
categories: [agents]
tags: [agents, google, formal-verification, multiagent, research, lean4]
params:
  author: AI Beat Desk
  summary: >-
    Google Research published results from Antigravity's /teamwork, a
    multi-agent system that proved the 13-year-old Courtade-Kumar conjecture,
    wrote 800,000 lines of safe Rust for the Zircon microkernel (which now
    boots on arm64), formalized the Weil conjectures in Lean 4, and hit 95%
    exact-match on NMR structure determination. The common thread is not raw
    capability but architecture: parallel exploration paired with automated
    falsifiers at every step.
---

The [Antigravity /teamwork post](https://research.google/blog/advancing-science-engineering-with-antigravitys-teamwork-feature/) from Google Research on November 13 is one of those announcements that benefits from slowing down. It is not a benchmark paper or a model launch. It is an enumeration of things that got done — in mathematics, formal verification, systems engineering, and chemistry — with a multi-agent system the team is now opening to outside researchers.

The core mechanism is straightforward: parallel exploration with automated falsifiers. Each candidate proof, each patch, each function gets checked immediately by an independent verifier — Lean 4 for proofs, unmodified test suites for code, a physics-informed simulator for chemistry spectra. Agents are partitioned so each sees only what it needs, limiting attention dilution on multi-day tasks. Critically, critics and auditors run from clean contexts; they do not inherit the proposer's reasoning history. This turns out to matter a lot. Reviewers who cannot see how a proof was constructed are less likely to replicate the same errors.

The math results are specific enough to be useful. The [Courtade-Kumar conjecture](https://www.emergentmind.com/open-problems/courtade-kumar-conjecture-most-informative-boolean-function), from 2013, asks which Boolean functions maximize mutual information with a noisy copy of their inputs; it was open for over a decade. The agents reduced it to a 14-variable optimization problem, then closed it in full generality. Near-optimal bounds for adaptive statistical queries — a problem from 2014 — fell the same way, and the new algorithm happens to also give a near-space-optimal streaming result. A new O(n log n) integer multiplication algorithm was produced and validated against GMP bit-for-bit up to a billion bits.

The systems work is harder to dismiss than most benchmark numbers. 800,000 lines of safe Rust were added to the Zircon microkernel. The code passes kernel unit tests and boots on arm64. The [libgav1](https://chromium.googlesource.com/codecs/libgav1/) AV1 decoder was rewritten in roughly 170,000 lines of Rust — it started four times slower than the original C++, then ended up 20–30% faster than the hand-tuned C++ and assembly on x86_64 and ARM. That is not an AI-generated draft waiting for human review; that is a shipped, outperforming replacement.

For formal verification, the team produced a fully autonomous Lean 4 formalization of the Weil conjectures — roughly 1.8 million lines from about 28,000 subagent invocations. It also formalized Hilbert's tenth problem over general rings of integers (919,000 lines), and found and fixed a gap in an informal proof of the 3D Bochner–Riesz conjecture during the process.

The chemistry result stands out for a different reason. 30 specialized agents in a self-correcting loop with a physics-informed NMR simulator reached 95% top-1 exact match on complex benchmark molecules — nearly double what a single agent achieves. The structure determination problem has practical implications well beyond academic chemistry; identifying molecular structures from spectroscopic data is routine industrial work.

The design principle running through all of this is not "smarter model." It is "cheap falsification." When you can tell instantly whether a proof compiles, a test passes, or a spectrum matches, you can run thousands of exploration threads in parallel without worrying about hallucination compounding across steps. The model only needs to be good enough to generate plausible candidates; the verifier decides what survives. This scales differently than prompting a single model more carefully.

Access to /teamwork and the other Antigravity features is open now for Gemini 4 users, with an application for 10,000 academic researchers to get six months of free access to the highest tier.
