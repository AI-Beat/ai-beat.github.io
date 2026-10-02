---
title: "Claude Code Grows a Mod System"
date: 2026-10-02T22:00:00+02:00
draft: false
slug: claude-code-mods
categories: [tools]
tags: [claude-code, plugins, agents, typescript, codex]
params:
  author: AI Beat Desk
  summary: >-
    Claude Code 2.1.287 ships mods: in-process TypeScript modules that can
    intercept, rewrite, and answer agent events before they reach the model.
    Unlike shell-script hooks, mods run inside the session, keep persistent
    state, and carry a rich sandbox API covering UI, tool routing, secret
    redaction, and custom slash commands. OpenAI Codex has had lifecycle hooks
    for a while — the two approaches reveal different theories about where
    agent customization should live.
---

Claude Code 2.1.287, released today, ships [mods](https://x.com/ClaudeDevs/status/2105721434807083061): TypeScript (or JavaScript) modules that run inside Claude Code and hook into its event loop. You write a `register(on, options)` export, list your handlers in `hooks.json`, point `plugin.json` at it, and install via `/plugin`. The module loads once when the session starts and stays alive for its duration — it can keep state across events.

That persistence detail matters. Most agent customization systems work as shell scripts triggered at lifecycle points: the agent fires an event, the shell script runs, exits, and the result flows back in. Claude Code mods run differently — they're in-process, loaded once, with full access to a sandbox API (`$`) covering `ui`, `session`, `state`, `store`, `fs`, `tool`, `command`, `model`, and HTTP.

The event model has three operations: **observe** (call `next(e)`, inspect results), **rewrite** (modify event data before forwarding), and **answer** (return a response without calling `next()` at all, intercepting rather than observing). That last one is the interesting one: a mod can answer a tool call, satisfy a slash command, or respond to a permission request without the underlying system ever seeing it.

Concretely, the launch examples show what this unlocks:

- **Secret redaction**: intercept tool output, strip credentials before they enter the context window
- **Model routing**: answer certain tool calls by forwarding to a different model rather than letting Claude handle them
- **UI extension**: add custom panes alongside the transcript, bands above the prompt, interactive buttons and tabs — components that share session state with your hooks
- **Slash commands**: register `/mycommand` that runs your TypeScript function instead of a Claude turn
- **Permission gates**: approve or deny permission requests with custom logic rather than falling back to the default prompt

Four mods ship built-in: a `/diff` command and an `AGENTS.md` loader among them. Community demos that appeared within hours included Mermaid diagram rendering inline, a running tool-call counter displayed in a custom band, and a pane that shows test results live as Claude runs them.

The dispatch budget is 10 seconds per event, and mods can't break out of the sandbox. The `fs` and `http` surfaces exist but are constrained to what the session's permissions already allow.

**How does this compare to Codex?**

OpenAI's [Codex hooks](https://developers.openai.com/codex/hooks) cover similar lifecycle points — `PreToolUse`, `PostToolUse`, `PermissionRequest`, `UserPromptSubmit`, `Stop`, and others, with an `Interrupt` event added in v0.150.0. Hooks can run shell scripts or invoke MCP tools, and as of v0.148.0 they can run asynchronously. They're a mature, capable system: you can intercept and modify agent behavior with them.

The structural difference is where the code runs. Codex hooks fire external processes. Claude Code mods run inside the session. This means mods can hold state between turns, rerender UI on every event, and intercept-and-answer rather than just observe-and-side-effect. In exchange, you're now running TypeScript inside a process you don't fully control rather than a shell script you own completely.

Codex also has a separate [plugin extensions system](https://openai.com/index/devday-2026-recap/) for adding sidebar apps, file viewers, and conversation panels — that's more UI-focused and MCP-based. Claude Code collapses both layers into the same module.

The interesting design question is where agent customization should live. The Codex model favors loose coupling: hooks are external, composable, language-agnostic within the shell. Claude Code's model favors tight integration: mods can reach directly into the session's state and UI because they run inside it. Both are reasonable choices; they'll produce different ecosystems. The Codex approach makes mods easier to audit and sandbox; the Claude Code approach makes stateful UI and complex interception possible without serializing everything through stdin/stdout.

For teams using Claude Code heavily, the practical implications are immediate: secret redaction from tool output is the obvious first deployment, followed by routing expensive reasoning calls to cheaper models. The custom UI surface is harder to use well but has clear value for any workflow that generates visual output — test results, diffs, diagrams — and wants them shown without a context-window round-trip.
