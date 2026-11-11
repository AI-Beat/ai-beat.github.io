---
title: "AI Writes Six Million Lines of Proof"
date: 2026-11-11T06:16:08+0000
draft: false
slug: alphaproof-code-signal-formal-verification
categories: [security]
tags: [formal-verification, deepmind, cryptography, lean, agents]
params:
  author: AI Beat Desk
  summary: >-
    Google DeepMind's AlphaProof Code used Gemini-powered agents to formally
    verify Signal's end-to-end encryption implementation at the compiled-code
    level, producing over six million lines of machine-checked Lean proofs in
    four weeks. The result is the first formal verification of a widely
    deployed cryptographic protocol's actual implementation — not just its
    design — and it opens a path toward AI-assisted security guarantees for
    critical software at scale.
---

Formal verification has long occupied an uncomfortable position in software security: everyone agrees it's the gold standard, almost no one does it, because it requires rare expertise and ungodly amounts of time. A team at Google DeepMind just took a significant step toward changing that. Yesterday they published [AlphaProof Code](https://deepmind.google/blog/introducing-alphaproof-code/), a multi-agent system they used to formally verify the implementation of the Signal protocol — the encryption scheme protecting messages in Signal, Google Messages, and several other widely deployed apps.

The key word is *implementation*. Prior formal verification work on cryptographic protocols typically targets the protocol design — a mathematical abstraction of how the system should behave. What AlphaProof Code verifies is the compiled code as it actually runs on devices. That gap between design and implementation is where most real-world cryptographic vulnerabilities live: the design is correct, but a subtle coding mistake, a compiler behavior, or an unexpected memory layout creates a discrepancy. Bridging that gap is significantly harder, and it's what makes this work notable.

The pipeline runs in two stages. First, a mix of models translates Signal's security goals — things like forward secrecy and post-compromise recovery — into formal specifications in [Lean](https://leanprover.github.io/), the proof assistant developed at Microsoft Research and now maintained by the Lean community. Human experts review each specification before it moves forward. Then agents powered by Gemini Carbon write the proofs themselves, producing over six million lines of Lean that Lean's kernel checks mechanically. The agents flag roughly 1% of the codebase as requiring human review; the rest is verified automatically.

Four weeks total, start to finish. The post notes that comparable work has historically taken expert teams years, and that's not far off — projects like the [seL4 microkernel verification](https://sel4.systems/Info/FAQ/proof.html) or [CertiKOS](http://flint.cs.yale.edu/certikos/) each represented multi-year efforts by specialized teams.

The team ran a sanity check that's worth highlighting: they reintroduced two previously fixed bugs into an isolated copy of Signal's codebase and ran the system against the modified code. In both cases, the agents produced a machine-checked proof that the buggy version violated a security property. This is the kind of test that distinguishes "we proved things" from "we proved the right things" — finding known bad states is a meaningful signal that the specifications are capturing real constraints rather than trivially true statements.

Google DeepMind is open-sourcing the formal verification artifacts, though not, as far as the announcement says, the full AgentProof infrastructure. That matters for reproducibility and trust — Lean proofs are checkable by anyone with Lean installed, so the artifacts themselves are verifiable independent of DeepMind's claims about the process. The announcement from Signal's CTO Ehren Kret frames this as a major confidence boost for the protocol's privacy guarantees, which is fair; having machine-checked proofs of your security properties is categorically better than not having them.

A few caveats worth keeping in mind. The formal specifications were written with human review, and a spec can be both machine-verified and subtly wrong if it doesn't capture the full threat model. The system also covers Signal's implementation, not every downstream deployment or integration layer. Formal verification proves properties of a fixed codebase; every future commit needs to be re-verified, or at least verified incrementally. The DeepMind post hints at automating that incremental verification, but it's not described as solved.

Still, the direction is clear and the trajectory matters. If AI agents can reduce the time cost of formal verification by one or two orders of magnitude, the economics change entirely. Projects that couldn't justify a multi-year verification effort might justify a few-week one. Medical device firmware, power grid controllers, and banking infrastructure all sit in that category — high-stakes, rarely formally verified, and increasingly targeted by sophisticated attackers. AlphaProof Code is a research result for now, but the problem it's attacking is real, and the approach is credible.
