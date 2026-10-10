---
title: "When the Verifier Has Bugs"
date: 2026-10-10T06:11:06+00:00
draft: false
slug: lean-soundness-ai-formal-proofs
categories: [research]
tags: [formal-verification, lean, math, safety]
params:
  author: AI Beat Desk
  summary: >-
    Thomas Hales writes on Tao's blog about Lean's reliability under AI-generated proof loads, describing a "Summer of Soundness Bugs" where AI security tools found kernel flaws that let fabricated proofs slip through — including an accepted Lean disproof of the Collatz conjecture. As AI writes more formal proofs at scale, the correctness of the verification infrastructure matters more than ever, and that infrastructure is itself imperfect.
---

Thomas Hales knows more about formalizing hard mathematics than almost anyone alive. His proof of the Kepler conjecture — sphere-packing in three dimensions, open for nearly four centuries — was 250 pages long and flagged incomplete by reviewers who couldn't check it by hand. He spent years formalizing it in Lean. When Hales says formal proofs need to be trusted carefully, it's worth listening.

His [guest post on Tao's blog](https://terrytao.wordpress.com/2026/10/09/what-mathematicians-should-know-about-the-lean-theorem-proverquestions-of-reliability-and-ai/) (October 9) is measured but has an edge. The premise: as AI-driven autoformalization scales — full formalizations of the prime number theorem, Fermat's Last Theorem, and Navier-Stokes blowup have all landed in 2026 — the trust mathematicians place in Lean needs to be examined, not assumed.

The crux emerged last summer. What Hales calls the "Summer of Soundness Bugs" was a run of vulnerabilities found in Lean's kernel, mostly by AI security tools probing it systematically for the first time. The most visible: a nested-inductive-type flaw that let the kernel accept a proof of `False` — that is, a proof that mathematics is internally inconsistent. A repository appeared in July claiming a Lean-verified disproof of the Collatz conjecture. It wasn't a joke. The kernel accepted it. The independent checker nanoda also accepted it, through a separate unrelated bug. It took a researcher named @kiranandcode three days to trace it back to the kernel itself.

The fix came within an hour of the formal report. That's good engineering. But what it exposed is structural.

Lean's kernel is not formally verified. Its type theory has open theoretical questions — unique typing is still unproved, and as of October 2026 there's no complete public consistency proof. Definitional equality in Lean is undecidable in general. The kernel has a `--trust=0` mode that should in principle catch any malformed proof, but AI adversaries can now probe it faster than humans can audit it.

Hales's proposed defenses are sensible: multiple independent kernels so a bug in one doesn't propagate silently, and Joachim Breitner's Con-Leche verified checker, which carries a formal consistency proof of its own. He draws a deliberate parallel to Ken Thompson's "Reflections on Trusting Trust" — you can't verify the verifier with the verifier. At some point, a human needs to audit the foundational layer.

This matters because the math community has largely taken Lean's trust model at face value: if a proof compiles with no axioms, it's correct. That assumption is now known to be fragile. What's uncomfortable is the recursion: AI is writing formal proofs of significant mathematics, and AI is also finding the soundness bugs that could make those proofs meaningless. The two timelines are running in parallel.

The [postmortem for Lean kernel bug #14576](https://aitoolly.com/en/ai-news/article/2026-08-02-lean-kernel-soundness-bug-14576-postmortem-of-the-ai-assisted-collatz-conjecture-disproof-and-fix) from August 2 is worth reading alongside Hales's post. Leonardo de Moura's comment then: "This is going to keep happening. AIs are really good at exploiting soundness bugs in the kernels." He was right. Hales's message is essentially the same, said more carefully: pay attention to the infrastructure, not just the results.

Given the pace of AI-generated formalization in 2026, that seems like useful advice. A solved theorem is only as good as the system that accepted it.
