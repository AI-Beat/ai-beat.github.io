---
title: "OpenCode Grows a Nervous System"
date: 2026-10-15T06:10:00+00:00
draft: false
slug: opencode-2-code-mode
categories: [tools]
tags: [open-source, coding-agents, mcp]
params:
  author: AI Beat Desk
  summary: >-
    OpenCode 2.0 ships a rebuilt architecture: a single shared background server
    that connects the TUI, web interface, and desktop app; Code Mode — a native
    JS interpreter that lets the agent chain MCP tool calls as scripts rather
    than one-by-one; and a hot-reloadable plugin system stable enough that the
    agent can write plugins for itself mid-session.
---

[OpenCode 2.0](https://anoma.ly/notes/opencode-2-0/) landed this week, and the jump from 1.x is more structural than feature-list. The visible additions — Code Mode, background agents, a plugin SDK — are interesting, but they're downstream of a single architectural decision: everything now connects to a shared background server that starts automatically and stays running.

In the 1.x design, the TUI, the web interface, and the desktop app each had their own process model. In 2.0 there's one service, and the interfaces are clients of it. The agent can reach the API directly — for example, to move a running session into a git worktree. Scripts can talk to it through `@opencode/client` or `opencode api` commands. This sounds like plumbing, but it changes what's possible: state persists across surfaces, work continues after a terminal window closes, and the SDK (`@opencode/sdk`) is just the same code that powers the CLI, available as a library. Liquid assets like Cloudflare Durable Objects are listed as a target environment.

The feature that most changes the agent's interaction with tools is Code Mode. In `@opencode/codemode`, the agent writes a small JavaScript snippet — a real script, not a prompt describing tool calls — that chains MCP operations as a single unit. All connected MCP servers are opted in by default. The practical difference from calling tools one-by-one is that the agent can express conditionals, loops, and error handling inside the script rather than reasoning about them step-by-step across turns. A sequence like "list all failing tests, for each one open the relevant file and check the last git blame" becomes a short script instead of a half-dozen round trips.

The interpreter runs natively rather than inside a Node.js VM or QuickJS — which means fewer sandboxing constraints but also that the agent is writing code that executes with the process's full ambient authority. That's not new for coding agents; Claude Code does the same thing via bash. It's worth being aware of.

The plugin system is perhaps the most self-referential piece. More than 80 features in the codebase are built as internal plugins, TUI plugins are now stable, and hot-reloading works throughout. The stated capability — that "OpenCode can write a plugin for itself mid-session" — is technically correct given the architecture, but it's also a claim that should be tested rather than taken at face value. A coding agent that writes its own UI extensions and hot-loads them is doing something that has real potential for surprising behavior, in both directions.

Breaking changes from 1.x: plugins don't port over, the server API is completely different, and `tui.json` config moves to JSON. The upgrade path is `opencode upgrade --major` from the latest 1.x release.

The open-source coding agent space has moved fast enough that OpenCode doesn't get discussed much outside specific communities — the coverage tends to cluster around Claude Code, Cursor, and whatever just shipped from OpenAI. The 2.0 architecture is a reasonable bet on where things go: coding agents that run as persistent services rather than per-session CLIs, with a programmable middle layer between the model and the tool environment. Whether that plays out better here or somewhere else is unclear, but the design is coherent.
