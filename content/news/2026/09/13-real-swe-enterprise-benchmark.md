---
title: "The Enterprise Gap"
date: 2026-09-13T06:09:51+00:00
draft: false
slug: real-swe-enterprise-benchmark
categories: [benchmarks]
tags: [benchmarks, coding, agents, enterprise]
params:
  author: AI Beat Desk
  summary: >-
    Specific Labs released Real-SWE, a benchmark that tests frontier AI coding
    agents on licensed private production codebases — tasks with real business
    stakes like billing systems and tax compliance. The best model (Fable 5.1)
    resolves 38.8% of tasks. The most expensive model doesn't win by much over
    models costing a third as much per rollout, and failure rates range from
    61–84% depending on the model.
---

Most AI coding benchmarks share a structural property: the code is public. SWE-bench uses GitHub issues from open-source repositories. FrontierCode samples pull requests from known projects. These are sensible choices — you need reproducible, machine-verifiable tasks — but they create a selection effect. Public repos tend to be well-structured, well-documented, and maintained by people who write for external readers. Enterprise production codebases are often none of these things.

[Real-SWE](https://withspecific.com/benchmarks/real-swe), released by Specific Labs this week, takes a different approach. They licensed private production codebases from real companies — fintech platforms processing transaction volumes at scale, SaaS products with 200K+ users — and extracted genuine engineering tasks from them: fixing billing logic, managing tax compliance workflows, migrating customer data. Each task is evaluated using verifiers derived from the codebase's own existing test suite, run in isolated Harbor sandboxes.

The results are instructive in ways that headline numbers don't fully convey.

The top performer is Fable 5.1 at 38.8% resolution. GPT-6 Astra is at 33.8%, Gemini 3.8 Flash at 31.2%, GLM 5.3 at 28.8%, and the field drops off from there — Kimi K3 at 18.8%, GPT-5.6 Sol at 16.2%. The spread between first and last is substantial, but the more striking fact is the floor: every model fails between 61% and 84% of the time. There is no model that handles even half of real enterprise engineering tasks reliably.

The cost story is also interesting. Fable 5.1 runs at roughly $6.96 per rollout. Gemini 3.8 Flash costs about $2.50. The gap in resolution rate between them is 7.6 percentage points. Whether that 7.6pp is worth 2.8x the cost is a business decision, not an engineering one, but it suggests the frontier is more compressed than benchmark leaderboards imply — the best model isn't so far ahead that cost stops mattering.

Task variance is high. Some tasks resolved at 0% across all models ("Analytics stream reducer" apparently stumped everyone). Others reached 67.2% ("Multi-region sweep"). This isn't surprising — enterprise codebases accumulate idiosyncratic design decisions and undocumented dependencies — but it does mean that aggregate resolution rate is a coarse signal. A model that solves your specific class of tasks at 60% while failing on others at 5% could be more or less useful than its average implies.

The dominant failure mode across all models was missing requirements. Agents would produce plausible-looking code that passed some tests but overlooked specified behaviors. This pattern aligns with what you'd expect from models trained heavily on public code: public repositories reward correctness on documented behavior, and enterprise tasks often have the critical constraints buried in comments, internal wikis, or the institutional memory of the engineer who wrote the original system.

The benchmark has obvious limitations. It reflects Specific Labs' particular collection of enterprise customers, which may skew toward certain industries or company sizes. The tasks were extracted by Specific Labs rather than directly submitted by the engineers who owned them, which adds a layer of interpretation. And resolution rate against a test suite doesn't capture whether the agent's approach is maintainable, secure, or aligned with the codebase's architecture — only whether it passes the checks.

But the core finding holds: models that look competitive on public benchmarks cluster around a 60–73% failure rate on real enterprise work. The gap between what gets announced and what gets deployed is not closing as fast as the benchmark curves suggest.

