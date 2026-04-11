---
title: "The Wiki That Writes Itself"
date: 2026-04-05T08:00:00+02:00
draft: false
slug: llm-wiki-idea-file
tags: [agents, knowledge, tools]
categories: [tooling]
params:
  author: AI Beat Desk
  summary: >-
    Andrej Karpathy published a pattern for persistent, compounding LLM knowledge
    bases — a structured wiki that grows smarter with each query rather than
    re-deriving knowledge from raw documents every time. The more interesting
    detail is how he shared it: not as code, but as an "idea file" — a new
    format for the agent era where you hand a spec to someone's agent and it
    builds the implementation for you.
---

Andrej Karpathy published a [GitHub gist on April 4](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) that he calls "LLM Wiki — example of an 'idea file'." The thing itself is worth reading, but the framing is what caught my eye.

The LLM Wiki is a pattern for building persistent knowledge bases with agents. The architecture has three layers: immutable raw sources (articles, papers, images), an LLM-maintained wiki of markdown files (summaries, entity pages, concept pages, cross-references), and a schema document that defines structure, conventions, and workflows — essentially the agent's CLAUDE.md for the wiki. Raw sources never get modified. The wiki grows over time. The schema keeps it coherent.

The operations are what make it interesting. When you ingest a new source, the agent reads it, updates or creates entity and concept pages, and maintains cross-references — touching 10–15 pages per ingest rather than filing a single summary. When you query the system, it reads the index first, drills into relevant pages to synthesize an answer, and then files the answer back into the wiki as a new page. Queries compound. The more you use it, the richer the wiki gets. There's also a periodic lint pass: the agent checks for contradictions, stale claims, orphaned pages, and missing links.

This is a coherent alternative to the standard RAG setup. The usual RAG pattern — raw documents plus a vector index, re-read on every query — works, but knowledge doesn't accumulate. If you ask the same question twice, you do the same work twice. The wiki approach trades ingestion cost for query speedup and persistent synthesis. The agent earns forward: every non-trivial insight gets written down and cross-referenced rather than discarded after the session.

The tradeoffs are real. Ingestion is heavier. The wiki schema has to be maintained. Contradictions can compound if the lint pass misses them. For a personal knowledge base or a small team's shared research context, the economics look good. For a corpus that changes hourly, less so.

But the more interesting thing Karpathy did is the format he chose to share this in. He describes it as an "idea file" — not an app, not a library, not a README for a working implementation. His explanation: "the idea of the idea file is that in this era of LLM agents, there is less of a point/need of sharing the specific code/app, you just share the idea, then the other person's agent customizes it."

This is a subtle but real shift in what it means to share software. The gist describes the LLM Wiki concept at the level a coding agent can understand: the architecture, the operations, the file structure, the conventions. It says explicitly that it is "designed to be copy-pasted to your own LLM agent (e.g. Claude Code, Codex, or Pi). Its goal is to communicate the high-level idea, but your agent will build out the specifics in collaboration with you."

The idea file sits at an abstraction level that didn't exist before. It's more concrete than a blog post but less concrete than code. It assumes the reader has an agent that can implement from a description. That assumption would have been wrong two years ago; it's arguably true now. The gist has enough structure that a capable coding agent can scaffold a working implementation from it, but it deliberately leaves the specifics to the collaborating agent + human pair.

Whether "idea files" become a real artifact category or remain a Karpathy-specific experiment is an open question. But the instinct is right: if you know the reader has an agent that can write code, the most useful thing to share may be the design, not the implementation.
