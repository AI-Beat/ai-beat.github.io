---
title: "Agent Memory as a Deductive Database"
date: 2026-08-29T06:12:24+00:00
draft: false
slug: lemmalog-datalog-agent-memory
categories: [agents]
tags: [agents, memory, datalog, open-source, tooling]
params:
  author: AI Beat Desk
  summary: >-
    Jordy Zomer's Lemmalog treats LLM agent memory as a Datalog deductive database
    rather than a vector store: facts carry provenance, conclusions retract when
    their premises are invalidated, and incremental evaluation keeps per-query cost
    constant regardless of history length — yielding 45x token savings on LongMemEval.
---

Vector databases became the default answer to LLM memory somewhere around 2024 and mostly stayed there. The pattern is simple: embed past context, retrieve the most relevant chunks at query time, stuff them into the prompt. For many use cases it works fine. But for anything resembling an extended investigation — security research, debugging, literature review — it has a structural problem that Jordy Zomer names precisely in a [post published yesterday](https://pwning.systems/posts/llm-memory-program-analysis/): retrieval doesn't retract.

If an agent rules out a hypothesis early in a session and later retrieves a fact that depended on that hypothesis, a vector store has no mechanism to flag the dependency. The agent rediscovers the ruled-out path. This is the kind of quiet degradation that doesn't show up in short benchmark runs — it accumulates over the course of a multi-hour investigation, as each retrieved chunk re-introduces invalidated assumptions the model may have already worked through.

Zomer's insight is that this looks exactly like a problem program analysis already solved. Static analysis tools maintain facts with derivation chains, apply rules to compute closures, and invalidate dependent conclusions when inputs change. The semantics Datalog provides — stratified negation, fixpoint computation, provenance through semiring annotations — map directly onto what agent memory actually needs.

[Lemmalog](https://github.com/JordyZomer/lemmalog) is the result. The LLM interacts with it exclusively at the extraction boundary, asserting triples via a simple line protocol (`S --rel[conf]--> O`). The engine takes those triples and derives closures deterministically through user-defined rules. Every derived fact carries a proof tree: `why()` traces the chain back to source episodes, with cycle protection. When a premise is retracted, dependent conclusions update automatically through incremental seminaive evaluation.

The bi-temporal design is worth noting separately. Facts carry `valid_from` and `valid_to` timestamps, so the engine models both when something was observed and when it was true in the domain being analyzed. This matters for vulnerability research — "this binary has ASLR disabled" may have been true at observation time but false after a patch — and it's the kind of temporal nuance that disappears entirely in a vector store.

Retrieval is hybrid: BM25 plus entity-graph boosting plus budget-aware assembly. The entity resolution layer handles the alias problem that plagues any long investigation: if the agent refers to the same function as `handle_auth`, `handleAuth`, and `the auth handler` across separate observations, they need to collapse to a canonical node before rules can fire across them. Lemmalog does this through star-shaped aliasing and canonicalization.

There's also an MCP server exposing 12 tools — `lemmalog_observe` for ingestion, `lemmalog_query` for retrieval, `lemmalog_why` for provenance — so Claude and Kimi can use it directly from a Claude Code session.

The benchmark numbers: 0.463 F1 on LongMemEval (versus 0.712 for full-context prompting, though at 38× smaller context), 0.533 on LoCoMo, with particular strength on knowledge-update questions and adversarial queries that require invalidating previously-stated premises. The headline figure is **45× token savings** on LongMemEval while maintaining roughly two-thirds of full-context accuracy. Whether that tradeoff is acceptable depends entirely on what you're building — for sessions where the history is genuinely hundreds of hours long, "38× smaller context" matters more than the F1 gap.

The honest caveat from Zomer himself: most of the performance improvement comes from the extraction boundary, not the Datalog engine. Entity resolution quality, the precision of `LlmExtractor` assertions, and semantic retrieval calibration drive most of the gains. The engine's value is architectural: provenance, retraction, incremental evaluation, and hypothetical reasoning via `what_if` lookahead. Whether you need those guarantees depends on whether your agent is doing shallow retrieval or building a genuine world model of the thing it's investigating.

For the class of problems where investigation state *needs* to be a proper logical structure — security research, extended debugging, anything where conclusions depend on conclusions depend on premises — Lemmalog makes a strong case that Datalog's 40-year-old semantics are exactly what's been missing from the agent memory stack.
