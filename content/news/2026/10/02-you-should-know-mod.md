---
title: "A Second Pair of Eyes on Your Agent"
date: 2026-10-02T23:00:00+02:00
draft: false
slug: you-should-know-mod
categories: [tools]
tags: [claude-code, agents, safety, plugins, monitoring]
params:
  author: AI Beat Desk
  summary: >-
    Claude Code 2.1.288 ships a built-in mod called "You should know" — a side
    agent that watches your main session and speaks up when it notices something
    you or Claude might miss. It's opt-in, requires telemetry, and represents
    a specific bet on how to surface the class of problems that slip through
    when you're not watching the terminal.
---

One of the practical problems with agentic coding sessions is attention asymmetry. Claude Code is working — running commands, editing files, making decisions — and you're either watching every token or you've stepped away. When you're watching, the risk is low. When you're not, things can go wrong in ways you'll notice only later: a .env file read into context, a `git push --force` that looked like the right move at the time, an API call made to a live endpoint during what you thought was a dry run.

Claude Code 2.1.288, released today, ships a built-in mod that addresses this directly: **You should know** (`cc-plugin-you-should-know@builtin`). You enable it with `/plugin enable cc-plugin-you-should-know@builtin`. It runs a side agent alongside your main session that watches what's happening and surfaces observations when something warrants your attention.

It's the first first-party use of the new [mods system](https://ai-beat.github.io/news/2026/10/claude-code-mods/) — which itself launched yesterday in 2.1.287 — and it demonstrates what the system can do that shell hooks cannot: the side agent maintains its own session state across the whole conversation, observes every event in the main session's event loop, and can write into the transcript when it decides something is worth flagging. Unlike a permission hook that fires synchronously on each tool call, "You should know" is asynchronous and opinionated: it decides when to speak, not every time.

The official description is deliberately minimal — "a side agent that watches your back and flags things you or Claude might miss" — so the specific categories it monitors aren't fully documented. Community-built observers like [lookout](https://github.com/ndanglin11-afk/lookout) (which does something similar with transcript scanning) give a reasonable proxy for what this kind of watcher targets: secret exposure (reading or writing `.env` files, inline API keys appearing in tool output), destructive operations (recursive forced deletes, database drops, force pushes to protected branches), dangerous install patterns (curl piped to shell), and outward-facing actions (sending email, creating calendar events, publishing deployments). Whether "You should know" monitors all of these isn't confirmed, but these are the categories that cause real post-mortem conversations.

The telemetry requirement is worth noting. The mod is available only in first-party sessions with telemetry enabled. That's a meaningful constraint: users who have opted out of telemetry for privacy reasons won't have access. The tradeoff Anthropic is implicitly making is that the side agent's training and improvement depend on seeing real session data, and that dependency limits who can use it. A community mod without that constraint could offer similar observability — the hooks exist — but would lack whatever signal the Anthropic-trained observer brings.

The broader pattern here is observer agents: a subagent whose only job is to watch another agent. The community has been building these manually (the lookout repo above, the multi-agent observability tools that appeared after swarm orchestration became common). Making one a first-party built-in normalizes the pattern and sets an expectation that running one agent means you probably want another watching it.

There's an obvious philosophical question: should the observer be the same model as the observed, and can it catch failures the main agent is prone to? A critic trained on the same base might share the same blind spots. The answer probably depends on what category of issue you're trying to catch. For mechanical things — a destructive command, a secret in output — a rule-based approach works fine and the model doesn't matter much. For subtler judgment calls — "this action seems outside the scope of what the user asked" — you want a model that reasons about intent, and it becomes a real question whether it can do better than the model it's watching.

For teams running Claude Code at any scale: enable this and see what it flags. The worst case is it's noisy; you disable it. The better case is it catches the class of quiet mistakes that don't announce themselves until later.
