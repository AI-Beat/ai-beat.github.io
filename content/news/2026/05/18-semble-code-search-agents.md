---
title: "The Context Budget Your Agent Wastes on Grep"
date: 2026-05-18T06:07:00+00:00
draft: false
slug: semble-code-search-agents
categories: [tooling]
tags: [agents, code-search, mcp, retrieval, token-efficiency, tools]
params:
  author: AI Beat Desk
  summary: >-
    Semble (v0.1.7, May 12) is a code search library for AI agents that uses
    ~98% fewer tokens than grep+read while matching 99% of the retrieval quality
    of much heavier transformer-based approaches. It indexes a repository in
    263ms and answers queries in 1.5ms on CPU, ships as an MCP server for Claude
    Code, Cursor, and Codex, and requires no API keys, GPU, or external services.
    The design bets that static embeddings plus BM25, fused carefully and
    reranked with code-specific signals, are almost as good as a code-specialized
    transformer — and orders of magnitude cheaper to operate.
---

There's a quiet tax on every coding agent session that isn't usually counted in the benchmark. Before a model can fix a bug or implement a feature, it has to find the relevant code. The naive approach — grep across the repo, read the matching files — works, but it burns context fast. A moderately sized codebase can easily blow 50–100K tokens on file discovery before the actual reasoning starts. For agents running inside tight context windows, or making many sequential tool calls, this is a real operational cost.

[Semble](https://github.com/MinishLab/semble), released to v0.1.7 on May 12 by MinishLab (Thomas van Dongen and Stephan Tulkens), is a purpose-built fix for this. The headline number is striking: 98% fewer tokens than grep+read, at 94% recall with a budget of only 2K tokens, compared to grep's 85% recall at a full 100K-token context. You're getting better retrieval quality while using almost none of the budget.

## How It Actually Works

The architecture is deliberately un-exotic. Semble doesn't try to fine-tune a large code model or build an embedding server that costs money to run. It uses [Model2Vec](https://github.com/MinishLab/model2vec) — specifically the `potion-code-16M` static embedding model — for semantic retrieval, and BM25 for lexical matching on identifiers and function names. Both run entirely on CPU.

Results from both retrievers are fused using Reciprocal Rank Fusion, a simple formula that combines ranked lists without needing calibrated scores from either system. The fused ranking is then reranked with a set of code-aware signals: a boost for chunks that contain definitions (the code unit most likely to be what the agent is looking for), penalty for test files and legacy directories, identifier stem matching, and file coherence weighting to group related chunks from the same file.

None of these components are new. What's notable is the combination: static embeddings are 200× faster to index and 11× faster to query than a code-specialized transformer like CodeRankEmbed, yet Semble achieves NDCG@10 of 0.854 versus CodeRankEmbed's 0.862. The gap is essentially nothing in practice. Indexing an average repository takes 263ms; answering a query takes 1.5ms. Both run on the CPU in your laptop.

## The MCP Angle

Semble ships as an MCP server, which means it's a drop-in tool for Claude Code, Cursor, Codex, and any other MCP-compatible agent. The agent calls `semble_search(query)` instead of reaching for shell-based grep, and gets back a focused set of code chunks rather than raw file contents. The token reduction flows directly from this: instead of the agent reading whole files to find a function definition, it gets just the relevant chunk.

This is a useful pattern beyond the immediate tool. The MCP ecosystem is maturing past the point where "give the agent a shell" is the default integration path. Specialized retrieval tools — narrower, faster, and token-efficient — fit naturally into agents that need to reason over large codebases without exhausting their context window on file navigation.

## What It Doesn't Do

Semble is a retrieval tool, not a comprehension tool. It finds relevant code chunks; the agent still has to read and reason about them. If the query is vague or the relevant code is spread across many files in non-obvious ways, the retrieval will surface imperfect chunks and the agent will need to do follow-up searches. That's inherent to the problem, not a flaw in Semble specifically.

The project is also early-stage — v0.1.7 released six days ago, built on eight prior releases over a short window. There's no discussion yet of how it handles monorepos, polyglot codebases, or repositories with unusual structure. The benchmark is NDCG@10 on code search, which doesn't directly measure what matters most in agent workflows: does it find the right thing on the first try, consistently, so the agent doesn't need to loop?

Still, the tradeoffs here are well-considered. Static embeddings are reproducible, deterministic, and free to run. The index is fully local and doesn't require a running server. The approach sidesteps all the operational overhead that comes with running a neural code search system. For teams building agents that need to work with large codebases on a token budget, it's the kind of tool worth dropping into the MCP config and measuring.
