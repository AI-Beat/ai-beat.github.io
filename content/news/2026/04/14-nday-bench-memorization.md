---
title: "The Vulnerability Benchmark That Knows What You've Already Read"
date: 2026-04-14T06:10:28+00:00
draft: false
slug: nday-bench-memorization
categories: [security]
tags: [benchmarks, security, evaluation, LLM]
params:
  author: AI Beat Desk
  summary: >-
    N-Day-Bench, a new benchmark from Winfunc Research, tests frontier LLMs
    on finding real vulnerabilities disclosed only after each model's knowledge
    cutoff — closing the memorization loophole that undermines most security
    evals. The April 13 run shows GPT-5.4 clearly ahead of the pack, with
    GLM-5.1 and Claude Opus 4.6 clustered close behind and Gemini 3.1 Pro
    trailing by 15 points. The methodology is the interesting part.
---

There's a quiet problem with most AI security benchmarks: they test whether a model can remember a CVE, not whether it can find one.

If your eval set consists of known vulnerabilities from code that existed during the model's training window, the model may have literally seen the bug — in a commit message, a security advisory, a blog post. Scoring 70% on such a benchmark says something about training data coverage, not about security reasoning. The two are easy to conflate.

[N-Day-Bench](https://ndaybench.winfunc.com), from Winfunc Research, is built around a simple fix for this problem: only test on vulnerabilities that were publicly disclosed *after* the model's knowledge cutoff date. An N-day is a vulnerability where N is any positive integer — it's been disclosed, but the model couldn't have seen it during training. The benchmark pulls from GitHub's security advisory database, scans 1,000 advisories per run, accepts the ones that meet quality criteria (47 made the cut in April), and then runs each model against the same harness with the same context.

The April 13 run tested five models:

| Model | Score |
|---|---|
| GPT-5.4 | 83.93 |
| GLM-5.1 | 80.13 |
| Claude Opus 4.6 | 79.95 |
| Kimi K2.5 | 77.18 |
| Gemini 3.1 Pro | 68.50 |

The gap between GPT-5.4 and the cluster of GLM-5.1 and Claude Opus 4.6 is about 4 points — meaningful but not dramatic. The more surprising result is Gemini 3.1 Pro sitting 15 points behind the rest. Gemini has generally performed well on code and reasoning tasks, so the gap is worth watching as more runs accumulate. Single-run numbers on a 47-case eval can move significantly; the benchmark's monthly cadence is designed to smooth this out over time.

The methodology matters for another reason beyond just avoiding memorization. Most security evals require annotators to construct synthetic vulnerability scenarios, which introduces its own bias — the bugs tend to look like the kind of bugs security researchers find interesting and write papers about. N-Day-Bench's source is GitHub advisories across real open-source projects, which means the distribution of bug types reflects what's actually getting found and disclosed, including a lot of unglamorous SQL injection, path traversal, and dependency confusion issues. That's closer to what a practical security tool would actually encounter.

The Hacker News thread surfaced a few open methodological questions. There's no false-positive rate measurement yet — models aren't tested on functions that don't contain vulnerabilities, so we don't know how often they hallucinate findings. The creator acknowledged this as an important gap and committed to adding it in a future run. The 47-of-1000 acceptance rate also raises a question about what's being filtered out: if the quality bar is high enough that 95% of advisories are rejected, it's worth understanding whether the accepted cases skew toward particular vulnerability classes or codebases.

These are tractable problems. The core insight — that you can sidestep the memorization problem by dating your test cases carefully — is clean and worth more attention than it's gotten. Most of the high-profile security capability claims from the past year have been evaluated on benchmarks that didn't control for this. N-Day-Bench isn't the last word on LLM security capability, but it's a more honest question than most.

The benchmark runs on a monthly cadence. All model traces are publicly browsable.
