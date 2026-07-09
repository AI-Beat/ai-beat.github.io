---
title: "The Ruler Is Broken"
date: 2026-07-09T06:10:51+00:00
draft: false
slug: swe-bench-pro-broken-databricks-evals
categories: [research]
tags: [evals, benchmarks, coding-agents, databricks, openai]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI's audit of SWE-bench Pro finds roughly 30% of tasks are broken, just
    months after SWE-bench Verified was retired for similar reasons. On the same
    day, Databricks published results from an internal benchmark built on real
    merged PRs — test execution, not LLM judges, no contamination. The two
    announcements together mark a quiet turning point in how serious users of
    coding agents think about evaluation.
---

For the past two years, SWE-bench has been the closest thing the coding-agent field has to a shared standard. Labs cite it in blog posts, leaderboards rank by it, VCs ask about it in due diligence. So [OpenAI's announcement yesterday](https://openai.com/index/separating-signal-from-noise-coding-evaluations/) is worth sitting with: their audit of SWE-bench Pro's 731-task public split found between 27% and 34% of tasks are broken.

That number comes from two sources. An automated investigator-agent pipeline flagged 200 tasks (27.4%); a follow-up human annotation campaign by five experienced software engineers identified 249 (34.1%). The failure modes are depressingly mundane — hidden requirements the spec never mentions, tests that reject correct solutions because they're too strict, contradictory instructions that make any answer wrong. Not subtle edge cases: tasks where the stated goal and the grading criteria simply disagree.

This is the second time in recent memory that a SWE-bench variant has been retired for data-quality reasons. OpenAI [previously stopped evaluating on SWE-bench Verified](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/) after finding widespread contamination and flawed tests. SWE-bench Pro was supposed to be the cleaner replacement, drawn from more recent PRs to limit training-set overlap. The new audit suggests the replacement also drifted — not from contamination this time, but from the laborious, human-intensive work of actually verifying that each task is well-posed.

The advice OpenAI offers is reasonable but uncomfortable: model developers should manually review task failures and task successes, since passing a broken task doesn't mean your model solved it correctly. That's sound guidance. It's also a large ask at the scale that modern agent evals operate.

---

On the same day, [Databricks published results from a different kind of evaluation](https://www.databricks.com/blog/benchmarking-coding-agents-databricks-multi-million-line-codebase): one they built themselves from actual merged pull requests in their production codebase — multiple millions of lines spanning Python, Go, TypeScript, Scala, and half a dozen other languages. Correctness is decided by test execution, not an LLM judge. There's no contamination risk because the tasks have never been public.

The results from this benchmark are interesting independent of the methodology. GLM 5.2, Zhipu AI's open-weight model, lands statistically tied with Anthropic's Opus 4.8 on task quality — at $1.28 per task against Opus's $1.94. That's not a small gap; over tens of thousands of daily developer tasks it adds up. The model-plus-harness interaction was the other surprise: the same underlying model routed through different agent harnesses showed cost swings of more than 2× while maintaining comparable output quality. Harness efficiency, it turns out, is not a footnote to model capability.

The Databricks benchmark won't replace a shared public standard — by construction it can't, since it's internal. But it illustrates what rigorous coding-agent evaluation actually looks like when you care about the answer more than the leaderboard position: real tasks, real tests, real costs, and no synthetic stand-ins for the messy polyglot codebase your actual engineers work in every day.

The timing feels deliberate even if it isn't. When the field's shared ruler is demonstrably unreliable, companies that care about picking the right tools will build private measuring sticks. What remains to be seen is whether the broader research community converges on a successor to SWE-bench Pro before the leaderboard culture quietly fills the vacuum by pretending the old one still works.
