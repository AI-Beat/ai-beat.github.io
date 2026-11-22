---
title: "A Proof That Compiles"
date: 2026-11-22T07:00:00+00:00
draft: false
slug: riemann-hypothesis-claude-lean
categories: [research]
tags: [mathematics, formal-verification, lean, anthropic, research]
params:
  author: AI Beat Desk
  summary: >-
    Levent Alpöge, an Anthropic mathematician, released 175,187 Lean proof files
    on November 20 claiming proof of the Riemann Hypothesis and GRH, with Claude
    providing key contributions "near the finish line." The Lean FRO and Mathlib
    Initiative issued a confidence statement the following day. Full kernel replay
    is still running; no traditional peer review has occurred.
---

The Riemann Hypothesis is the kind of problem that generations of mathematicians have orbited without landing. Posed by Bernhard Riemann in 1859, it claims that all nontrivial zeros of the Riemann zeta function lie on the line \(\text{Re}(s) = 1/2\). Its truth would pin down the distribution of primes with unprecedented precision; hundreds of "if RH then..." theorems would become simply true. It sits on the Clay Institute's Millennium Prize list and, more to the point, on the shelf of problems that have resisted everyone who tried.

On November 20, Levent Alpöge — a number theorist and former Harvard Junior Fellow now at Anthropic — released a note claiming he and Claude had proved it. Not just the classical hypothesis but also the Generalized Riemann Hypothesis for Dirichlet L-functions. He made the claim in the most verifiable possible form: [175,187 Lean proof files](https://casrai.org/news/anthropic-levent-alpoge-claude-lean-riemann-hypothesis-claim-2026), machine-checkable from first principles.

Alpöge's note describes the collaboration precisely. He set the strategy; Claude pushed through the technically demanding sections. "Far outpaced me near the finish line," he writes. This is not the first time Claude contributed here: an earlier Claude-assisted result, formalized in Lean, showed that over two-thirds of nontrivial zeros lie on the critical line — an improvement on the best prior bound. That work prepared the ground.

What makes this claim unusual is what "verification" actually means in Lean. Lean is a proof assistant: it checks formal proofs against the axioms of mathematics, mechanically, without interpretation. When [MIT's Andrew Sutherland independently compiled the archive](https://www.scientificamerican.com/article/anthropic-mathematician-claims-to-prove-riemann-hypothesis-most-famous-problem-in-all-of-math/) and reported that all 175,187 modules pass with zero errors, that is not the same as a peer reviewer reading the argument. It means the proof object is internally consistent in a way a machine can certify. The [Lean FRO and Mathlib Initiative issued a joint confidence statement](https://startupfortune.com/lean-groups-certify-claude-assisted-riemann-hypothesis-proof-from-anthropic/) on November 21; two independent checkers accepted the exported proof.

The caveats matter. The community replay project, which independently rebuilds the proof from source rather than checking exported artifacts, was at around 59% of modules as of early November 22. Full kernel replay is still running. Alpöge himself writes that there are sections he doesn't fully understand — areas where "Claude is the expert." The extension to the broader Extended Riemann Hypothesis, beyond Dirichlet L-functions, remains unfinished. No traditional peer review has happened, and none will come quickly for a 175,000-file Lean archive.

A proof being formally correct in Lean does not automatically mean the mathematical community will accept it on the terms they care about. What mathematicians typically want from a proof is not just that it checks out, but that it carries a reason — that reading it teaches you something about why the result is true. A proof where the author admits they don't follow all of it raises genuine questions about comprehensibility, even if Lean's kernel says it compiles. This dynamic already came up with the Flyspeck project (formal verification of the Kepler conjecture), where the community took years to absorb a machine-checked result.

Still: a Lean proof of the Riemann Hypothesis that the Lean FRO and Sutherland both confirm is not nothing. If the kernel replay finishes cleanly and the statement-matching checks pass, what remains is a formal object certified to the axioms mathematicians accept. That will force engagement whether or not the broader community finds it intellectually transparent.

The more important signal may be the trajectory itself. Claude improving the zero-density bound on the critical line, then crossing all the way to a full claimed proof, over roughly a year — that traces a capability curve in mathematical research that is hard to dismiss regardless of how the peer review goes.
