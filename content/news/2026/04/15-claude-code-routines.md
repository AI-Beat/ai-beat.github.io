---
title: "Claude Code Gets a Cron"
date: 2026-04-15T06:19:02Z
draft: false
slug: claude-code-routines
categories: [tooling]
tags: [anthropic, claude-code, automation, agents, infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic shipped Claude Code Routines in research preview: saved Claude
    Code configurations that run autonomously on Anthropic-managed cloud
    infrastructure on a schedule, triggered by an API call, or fired by
    GitHub events. The pieces have been building toward this — long-horizon
    sessions, Managed Agents, the advisor tool — and cloud-scheduled unattended
    execution is the natural next step.
---

Anthropic quietly shipped [Claude Code Routines](https://code.claude.com/docs/en/routines) to research preview. The headline is simple: you can now define a Claude Code session — a prompt, a set of repositories, a collection of MCP connectors — and have it run automatically, on a schedule, via an API call, or in response to a GitHub event, on Anthropic's cloud infrastructure. No laptop open, no human in the loop.

That's a meaningful step. The trajectory from "interactive coding assistant" to "autonomous agent you schedule and forget" has been underway for a while. Managed Agents gave you persistent cloud sessions with tool access. The [advisor tool](https://ai-beat.github.io/news/2026/04/advisor-strategy-anthropic-api/) layered in a cost-efficient planning tier. Routines close the loop: the session itself becomes a scheduled job.

## What it actually is

A routine is a saved configuration: a prompt, one or more GitHub repos (cloned fresh at each run), environment variables, an optional setup script, and a set of connectors. Each run is a full Claude Code cloud session — it can execute shell commands, use skills committed to the repo, and call any connectors you've included. The three trigger types are:

**Scheduled** — standard cron-like recurrence (hourly, nightly, weekly, or a custom cron expression via `/schedule update` in the CLI). The minimum interval is one hour.

**API** — a per-routine HTTP endpoint with a bearer token. `POST` to it and a new session starts, optionally with a `text` field for run-specific context (alert body, failing log, whatever). The response includes a session URL you can open in a browser to watch in real time.

**GitHub event** — fires on any of 17 event categories: PR opened/closed/reviewed, push, release, issues, check runs, workflow dispatch, and more. Each matching event starts its own session. Filters narrow which events trigger the routine — author, label, base branch, whether it's a draft, whether it comes from a fork.

The permissions model is conservative by default. Routines can only push to `claude/`-prefixed branches unless you explicitly enable unrestricted branch pushes for a given repo. Each routine is scoped to your individual account (not shared with teammates), and anything it does through GitHub or connected services appears as you.

## The use cases worth thinking about

The documentation lists sensible examples — nightly backlog triage, bespoke PR review, deploy verification. These are all real. But the API trigger with context injection is the most interesting primitive.

The model here: your monitoring system fires an alert. Instead of paging an on-call engineer who then spends the first 20 minutes correlating the stack trace with recent commits, a webhook calls the routine's `/fire` endpoint with the alert body as `text`. The routine pulls the stack trace, looks at the relevant repo history, and opens a draft PR with a proposed fix and a link back to the alert. The on-call reviews the PR instead of starting from scratch.

```bash
curl -X POST https://api.anthropic.com/v1/claude_code/routines/trig_01ABCDEFGHJKLMNOPQRSTUVW/fire \
  -H "Authorization: Bearer sk-ant-oat01-xxxxx" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod. Stack trace attached."}'
```

The cross-SDK port example is also worth noting: a GitHub trigger fires on merged PRs in one SDK repo and automatically opens a matching PR in a parallel SDK in another language. That's the kind of tedious mechanical translation work that's low-risk to automate and high-cost to do by hand.

## What it's not yet

This is research preview with explicit caveats: behavior, limits, and the API surface can change. The beta header (`experimental-cc-routine-2026-04-01`) signals this is production-adjacent but not production-committed. The GitHub trigger has per-routine and per-account hourly caps; events beyond the limit are dropped silently. There's no event queue. You also can't configure GitHub triggers from the CLI yet — that requires the web UI.

The autonomy boundary deserves attention. Routines run with no approval prompts and no permission-mode picker. What the routine can reach is determined entirely by the repos, environment, and connectors you configure at setup time. That's the right design — approval prompts in unattended automation are pointless — but it means the scoping work has to happen before the routine runs. Claude may also take actions that appear as you in external systems. Worth being deliberate about what connectors you include.

Usage counts against your subscription the same as interactive sessions, with an additional daily cap on total routine runs. Organizations with extra usage enabled can run on metered overage; without it, runs are rejected when limits are hit.

The [documentation](https://code.claude.com/docs/en/routines) is thorough, including the API reference and examples for all three trigger types. Create routines at `claude.ai/code/routines` or via `/schedule` in the CLI.
