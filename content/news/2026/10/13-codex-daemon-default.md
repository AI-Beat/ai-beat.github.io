---
title: "Codex Grew a Daemon"
date: 2026-10-13T06:11:06+00:00
draft: false
slug: codex-daemon-default
categories: [tools]
tags: [tooling, openai, security, cli]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI's Codex CLI version 0.157.0 quietly started launching a persistent
    background process that self-updates hourly, runs from a copy of itself in
    ~/.codex/packages/, and doesn't stop when you close Codex or even run
    `daemon stop`. The HN thread surfaced it; the question it raises is whether
    AI coding tools are quietly becoming apps.
---

When you open a terminal and run `codex`, you probably expect a CLI that starts,
does work, and exits. Since version 0.157.0, that's not quite what happens.

An [open GitHub issue](https://github.com/openai/codex/issues/48195) documents the
behavior. Opening an interactive Codex session now launches `codex app-server --managed-daemon`
in the background. That process:

- Keeps running after you close all Codex windows, until you manually stop it or reboot.
- Copies the full Codex package (~374 MB) into `~/.codex/packages/` and runs
  from that copy — not from the binary npm installed.
- Starts a detached updater that checks for new releases 5 minutes after launch,
  then hourly, downloads updates, and restarts the backend. This can interrupt
  active sessions.
- Persists the environment from the session that started it — proxy settings, working
  directory — and applies those to later sessions.
- Continues running when you run `daemon stop`. Only the app server stops; the updater
  stays.

The release note that shipped this called it "Enabled automatic background-server
startup for eligible interactive sessions." That's technically accurate the way
"our product collects usage data" is technically accurate — true, but written to be
skimmed past.

## Why it matters

None of this is unusual behavior for a native desktop app. Slack does it. VS Code's
extension host does similar things. The problem is that `codex` presents itself as
a command-line tool, and command-line tools carry an implicit contract: they run
when you invoke them, they stop when they're done, and they don't make lasting changes
to your system without telling you.

The gap between that expectation and what's actually happening is real. You installed
a package via npm. What you got is a service that:
- Runs a different binary from what npm installed
- Self-modifies by fetching and applying updates from OpenAI's servers
- Runs even when you've closed it
- Inherits environment variables you probably set for a specific session

The auto-update piece is particularly notable. Hourly update checks mean the binary
running your agent workflows can change without a `npm install` from you. If the
update introduces a regression, the affected binary isn't the one you installed
and may not match any version in your package manager's lock file. That makes
debugging harder and supply-chain auditing essentially impossible.

The community response in the thread runs from "this is fine, apps do this" to
"this reads like malware." Both reactions are too strong. It's not fine — opt-in
exists for a reason — but it's also clearly not malicious. The more interesting
question is what it signals: AI coding tools are increasingly expecting to live on
your system as persistent services, not as commands you invoke and dismiss.

Microsoft Execution Containers [shipped last week](https://blogs.windows.com/windowsdeveloper/2026/10/07/microsoft-execution-containers-policy-driven-containment-for-ai-agents/)
with the explicit goal of giving you policy-level control over what agent workloads
can do to your system. That's the right direction. The Codex daemon is a reminder
of why.

The ask in the issue is reasonable: make auto-daemon opt-in, or at minimum prompt
before enabling it. A process that runs silently on your machine, updates itself
from the network, and outlives the session that started it should not be a default
that users have to discover from a GitHub issue thread.
