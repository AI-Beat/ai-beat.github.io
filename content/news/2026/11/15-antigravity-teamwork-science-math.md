---
title: "Agents in the Lab"
date: 2026-11-15T06:12:46+00:00
draft: false
slug: antigravity-teamwork-science-math
categories: [agents]
tags: [agents, formal-verification, math, gemini, multiagent]
params:
  author: AI Beat Desk
  summary: >-
    Google Research's November 13 post on Antigravity's /teamwork describes
    coordinated Gemini 4 Argon agents proving the Courtade-Kumar conjecture,
    producing a 1.8-million-line Lean formalization of the Weil conjectures,
    hitting 95% on NMR structure determination, and rewriting libgav1 into
    Rust 20–30% faster than the original C++. The results are hard to dismiss
    and harder to fully interpret.
---

There is a type of AI announcement that arrives dressed as a press release but reads like a research log, and [Google Research's post on Antigravity /teamwork](https://research.google/blog/advancing-science-engineering-with-antigravitys-teamwork-feature/) from November 13 is one of them. Written by Vahab Mirrokni, it describes what happens when you point a large coordinated swarm of Gemini 4 Argon agents at open problems in mathematics, formal verification, chemistry, and systems engineering. The results are specific enough to take seriously.

On the mathematics side: a system using /teamwork proved the [Courtade–Kumar conjecture](https://en.wikipedia.org/wiki/Courtade%E2%80%93Kumar_conjecture), a 2013 problem about which Boolean functions maximize mutual information with a noisy observation of their inputs that had resisted a decade of attempts, and formalized the proof in Lean 4. It closed a sample-complexity gap in adaptive data analysis open since 2014, settled a quantum query complexity question from the same year, and gave a deterministic algorithm for triangle detection resolving an open question from STOC 2024. Those are verified, citable claims, not approximate ones.

Formal verification is where the numbers become genuinely unusual. The post describes a framework called LEAP that produced a machine-checked formalization of the Weil conjectures—about 1.8 million lines of Lean—plus formalizations of Hilbert's tenth problem over general integer rings, a recent proof of finite-time blowup for the 3D Navier–Stokes equations, the Birkhoff conjecture, and the 3D Bochner–Riesz conjecture. Formalizing a deep theorem in Lean is not a task where a model can confabulate its way to success; a type-checker is the judge.

Chemistry: a 30-agent system with a physics-informed spectral simulator reached 95% top-1 exact match on NMR-based structure determination for benchmark molecules. That's the problem of deducing a molecule's structure from its nuclear magnetic resonance spectrum, which requires both pattern matching and chemical reasoning.

Engineering work is less dramatic in headline terms but perhaps more consequential in practice. The agents rewrote roughly 170,000 lines of libgav1—a C++ video decoder used at scale inside Chrome—into Rust, achieving bit-exact parity and then exceeding the original's performance by 20–30%. They also rewrote a portion of the Zircon microkernel (~800K lines to Rust), built custom TPU kernels that reduced HBM usage by up to 185×, and contributed a Borg scheduling change that cut TPU workload preemption by 24% on some hardware. These aren't demos; they're changes in production systems.

The right way to read this is not "AI solved all these problems alone." The post is careful about framing: human experts set the questions, verified results, and edited outputs throughout. What the system appears to be doing is collapsing the search phase of research—generating, filtering, and formalizing candidate solutions much faster than a human team could, while leaving the humans to steer and judge. The Courtade-Kumar proof, for instance, has human authors on record and was accepted for publication; the agents apparently helped find and verify the proof path rather than writing the paper.

That distinction matters, but it doesn't diminish the result. Research bottlenecks vary by field, and for a large class of problems in mathematics and formal verification, the bottleneck is exactly the kind of exhaustive search and mechanical verification that agents are good at. The Lean formalization work is a good example: producing 1.8 million correct lines of Lean requires almost no human insight once the high-level proof strategy is known—it requires tedious, bug-free mechanical translation that is precisely the kind of work a type-checker can audit.

The /teamwork feature is now available in Antigravity to external users on paid plans using Gemini 4 models. Google is also offering six months of free top-tier access to 10,000 academic researchers. That's worth flagging: if the results above hold up at that scale, a lot of research groups are about to have access to something meaningfully different from a chat assistant.

The honest uncertainty here is about peer review. Several of the mathematical results are described as accepted or submitted to venues—the post is not vague—but independent replication of agent-assisted proofs is an infrastructure problem that the community hasn't fully solved. Lean code is at least machine-checkable; the benchmark chemistry results are harder to independently verify from outside. That's a caveat, not a dismissal.
