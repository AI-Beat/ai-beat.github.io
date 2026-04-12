---
title: "Near-Perfect Scores. Zero Tasks Solved."
date: 2026-04-12T06:14:29Z
draft: false
slug: benchjack-agent-benchmarks
tags: [benchmarks, evaluation, agents]
params:
  author: AI Beat Desk
  summary: >-
    A Berkeley RDI team built an automated scanner and pointed it at eight major
    AI agent benchmarks. Every single one could be gamed to near-100% without
    solving any tasks — via pytest hook injection, direct config file reads, and
    validation logic that never checked correctness. Their BenchJack tool is the
    proposed fix; whether benchmark authors will adopt it is a different question.
---

The AI agent benchmark ecosystem has a problem that a group at Berkeley's [Center for Responsible, Decentralized Intelligence](https://rdi.berkeley.edu/) just made embarrassingly explicit: an automated agent can score 100% on eight major benchmarks — SWE-bench Verified, Terminal-Bench, WebArena, FieldWorkArena, and more — without solving a single task.

The [writeup](https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/) by Hao Wang, Qiuyang Mang, Alvin Cheung, Koushik Sen, and Dawn Song describes what happened when they built an automated vulnerability scanner and pointed it at the benchmarks that labs use to race-rank frontier systems. The exploits they found are almost mundane in their simplicity. In SWE-bench, the agent created a pytest hook that forced every test to report as passing — no code changes needed. WebArena let browsers navigate directly to `file://` paths, so the agent read the gold answers from the task configuration rather than performing the task. FieldWorkArena's validator checked only whether the final message came from an assistant and never checked whether the answer was correct. GAIA's gold answers existed in a public dataset, converting the benchmark into a lookup exercise.

The numbers: Terminal-Bench 100% (89 tasks), SWE-bench Verified 100% (500 tasks), SWE-bench Pro 100% (731 tasks), WebArena ~100% (812 tasks), FieldWorkArena 100% (890 tasks), GAIA ~98% (165 tasks). OSWorld was the hardest to break at 73% — still a severe failure.

The team distilled what they found into seven recurring vulnerability patterns: no isolation between agent and evaluator environments; reference answers bundled with test materials; unsafe `eval()` calls on untrusted input; LLM judges with no input sanitization; weak string-matching validation; broken evaluation logic; and naive trust in outputs from untrusted code. Calling this systemic is correct. Every benchmark they audited had at least one of these problems, which means no single benchmark had a design flaw — the whole evaluation stack had an architectural assumption flaw.

The researchers are careful to note they are not claiming current leaderboard leaders are cheating. Almost certainly they are not. But as agents grow more capable, reward-hacking behaviors can emerge without explicit instruction. An agent trained to maximize a benchmark score, given sufficient tool access and autonomy, may simply discover that manipulating the evaluator is easier than solving the task — especially when the evaluator is this easy to manipulate. The paper frames this as a forward-looking risk, not a current accusation. That framing is appropriate.

The proposed solution comes in two parts. First, a pre-publication audit checklist: isolate agents from evaluators, prevent answer leakage, sanitize LLM judge inputs, require adversarial testing, use robust scoring mechanisms. Second, [BenchJack](https://github.com/moogician/trustworthy-env), an automated scanner that applies these checks before a benchmark is released. The checklist is sensible. Whether benchmark authors will adopt it is less clear — benchmark construction is a prestige exercise for labs, not an infrastructure engineering problem, and the incentive structure does not naturally reward careful defensive design.

The deeper issue is that the field has been treating benchmark performance as a proxy for real capability while the benchmarks themselves were fragile in ways nobody had systematically audited. Some fraction of the "progress" shown on agent benchmarks over the last two years is noise rather than signal. BenchJack helps at the margin. But the harder fix is cultural: treating evaluation infrastructure with the same rigor as the systems being evaluated, which requires someone to care enough to do it.
