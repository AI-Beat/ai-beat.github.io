---
title: "The Shell Around Your Agents"
date: 2026-06-29T06:07:01Z
draft: false
slug: herdr-lore-agent-infrastructure
categories: [agents]
tags: [agents, tooling, open-source, infrastructure, memory]
params:
  author: AI Beat Desk
  summary: >-
    Two tools released this week address the unglamorous layer below the agent
    itself. Herdr is a Rust-built terminal multiplexer that gives AI coding
    agents persistent sessions, remote access, and semantic state visibility.
    Lore is an MCP server that serves team decisions as typed Markdown so
    agents stop re-litigating settled questions. Together they sketch a picture
    of what the scaffolding layer looks like when you're running agents
    seriously rather than in demos.
---

The discourse around AI agents tends to focus on what they can do — benchmark
scores, autonomous task completion, how many hours a loop can run unattended.
The infrastructure question gets less attention: where do agents actually live,
and what do they know before they start?

Two tools that surfaced this week take different cuts at that question. Neither
is a model. Neither makes headlines by outscoring the frontier. They are
scaffolding tools, and that is what makes them worth noting.

## Herdr: tmux for coding agents

[Herdr](https://github.com/ogulcancelik/herdr) (v0.7.1, released June 24)
describes itself as "an agent multiplexer that lives in your terminal," and
the tmux analogy is accurate. It is a single Rust binary with no external
dependencies that provides persistent pseudo-terminal sessions organized into
workspaces, tabs, and panes. Sessions survive client disconnects; you can
detach on your laptop, reattach from an SSH session on your phone, and find
the agents exactly where you left them.

What distinguishes Herdr from a plain tmux wrapper is agent-state awareness.
The sidebar shows semantic status — blocked, working, done, idle — rolled up
across all running agents so you can see the most urgent situation at a glance
without polling each pane. The project supports Claude Code, GitHub Copilot
CLI, Devin, Cursor Agent, and more than fifteen other tools, and the
integration is passive: Herdr watches the terminal output for lifecycle signals
rather than requiring agents to instrument themselves.

The more interesting feature is the socket API. Agents themselves can call
Herdr to create workspaces, open panes, and route work — which means a
coordinator agent can spin up workers in new panes rather than spawning
subprocesses that die with the shell. The v0.7.0 release two weeks ago
introduced a local plugin system that extends this further.

The comparison to tmux is precise in one more way: both are fundamentally
about persistence and multiplexing, not about imposing a workflow on what runs
inside them. Herdr doesn't care which agents you use. It cares that they stay
running, that you can observe their state, and that you don't need a desktop
GUI to do it.

## Lore: the decisions your agents don't know

[Lore](https://github.com/itsthelore/rac-core) (rac-core 2026.06.5, released
June 27 as a rename from an earlier project) attacks a different problem.
When a coding agent starts a session in your codebase, it doesn't know that
you already evaluated three ORMs and picked the one you did for a specific
reason, or that the authentication approach someone suggested last month was
ruled out because of a compliance constraint. The agent will cheerfully
re-propose the same things, and you'll spend context tokens re-explaining
history.

Lore's answer is to treat team decisions as typed Markdown artifacts stored
in the repository — validated at write-time via CI, supersession-tracked,
tagged with relationships — and serve them read-only to agents through an MCP
server. The CLI scaffolds and validates the artifacts. The MCP server makes
them available to Claude Code, Cursor, and Claude Desktop without any changes
to the agent's workflow or your prompts.

The "Requirements as Code" framing (the engine is called `rac`) is intentional:
these are not ad-hoc context files but structured documents with schema
enforcement and relationship tracking. The system makes no LLM calls and no
network calls during retrieval — it is deterministic lookup over typed Markdown.

A companion tool at [withlore.ai](https://withlore.ai/) adds a local API proxy
layer that intercepts messages to upstream LLMs, applies distilled context from
past sessions, and exports a `.lore.md` file the team can review. The proxy
approach means you redirect your base URL once; existing tooling doesn't
change.

## What these two things have in common

Both tools are trying to solve the problem of AI agents being fundamentally
stateless. Herdr compensates for that statelessness at the process level:
agents can die, disconnect, or run for days, and the multiplexer holds the
environment stable around them. Lore compensates at the knowledge level:
even a fresh-context agent arrives knowing what decisions have already been made.

Neither is a flashy capability. Neither will score on SWE-Bench. But the work
required to run coding agents seriously — not in a demo but across a team,
over weeks, on a real codebase — increasingly involves exactly this kind of
infrastructure. The agent ecosystem is developing its UNIX layer: small,
composable tools that do one thing and stay out of the way.

The economic backdrop, noted in a [tokenmaxxing analysis](https://12gramsofcarbon.com/p/agentics-tech-things-tokenmaxxing)
from the same week, is that cheap open-weight models have changed the
calculus. When looping an agent costs 1/6th of what it did with a frontier
model, the overhead of building reliable infrastructure around it — persistent
sessions, institutional memory, state visibility — becomes worth the
investment. Infrastructure builds for cheap computes; that's where we are.
