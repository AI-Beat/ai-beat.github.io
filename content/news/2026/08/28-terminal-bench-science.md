---
title: "Science Is Harder Than Coding"
date: 2026-08-28T08:15:00+02:00
draft: false
slug: science-is-harder-than-coding
categories: [research]
tags: [evals, benchmarks, agents, science]
params:
  author: AI Beat Desk
  summary: >-
    Terminal-Bench-Science v0.1 evaluates AI agents on 70 real computational
    workflows from active research labs across five scientific domains. Claude
    Opus 5 tops the leaderboard at 30%, well below the 50–80% range frontier
    models achieve on general coding benchmarks — a gap that says something
    useful about where the actual difficulty lies in scientific work.
---

[Terminal-Bench-Science](https://snorkel.ai/terminal-bench-science/) released its v0.1 leaderboard this week with 70 tasks drawn from actual researcher workflows — pipelines a practicing scientist ran, packaged into containerized environments with programmatic, pytest-based verification. Claude Opus 5 leads at 30.0% resolution. GPT-5.6 Sol clocks in at 22.4%, Fable 5 at 21.4%. The tasks span five domains: Life Sciences (19 tasks), Physical Sciences (17), Mathematical Sciences (17), Earth Sciences (8), and Engineering Sciences (9).

The comparison point matters: frontier models routinely hit 50–80% on general software engineering benchmarks. The 30% ceiling here is not explained by the coding difficulty per se — the same models write working code across a wide range of tasks. What is harder in scientific workflows is everything around the code.

A research computational task typically involves calling domain-specific tools with non-obvious configuration, managing intermediate files in formats that vary by field, interpreting outputs that require scientific context to evaluate, and recovering from partial failures without a clear error signal. A bioinformatics pipeline might need to align reads against a reference genome, filter by quality thresholds that are standard in the field but not in the documentation, and feed the result into a downstream tool that expects a slightly different file format. The code to do each step is not the hard part. Knowing what "done correctly" looks like at each stage is.

Terminal-Bench builds on the same containerized evaluation infrastructure that has been adopted by Anthropic, OpenAI, and Google DeepMind for internal evals. The key design choice is sourcing tasks from real research workflows rather than constructing them from textbook exercises. A task drawn from something a researcher actually did carries the full complexity of that work — edge cases, format quirks, and domain-specific conventions included. Evaluation is deterministic: the agent's output is checked programmatically, not rated by a human or another model.

The 30% result for Opus 5 is probably closer to an accurate measure of current capability on scientific tasks than anything derived from MMLU or GPQA, which test retrieval of domain knowledge rather than workflow execution. An agent that can answer questions about protein folding is not necessarily an agent that can run an AlphaFold pipeline against a novel input, parse the output confidence scores correctly, and flag structures below a quality threshold. These are different skills.

What the benchmark will not tell you, at v0.1 with 70 tasks, is whether 30% represents a ceiling or a calibration point. Seventy tasks across five domains means roughly 14 tasks per domain on average — not enough to characterize where within each domain the failures concentrate. The v0.2 release is targeted for October, with an open task-submission pipeline running in the interim. As the task count grows, the leaderboard should start revealing domain-specific patterns: which types of scientific computation agents handle competently, and which remain reliably broken.

The broader "AI for science" narrative tends to conflate different things — scientific knowledge retrieval, hypothesis generation, literature search, and computational workflow execution. Terminal-Bench-Science evaluates the last of these, which is arguably the most tractable near-term target and also the one where current agents fall shortest of the marketing language. Thirty percent is a real number, and it is a more useful starting point than aspirational claims about AI accelerating scientific discovery.
