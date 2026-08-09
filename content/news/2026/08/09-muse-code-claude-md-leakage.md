---
title: "Muse Code Reads Your CLAUDE.md — and Sends It to Meta"
date: 2026-08-09T06:30:00+00:00
draft: false
slug: muse-code-claude-md-leakage
categories: [security]
tags: [muse-code, meta, privacy, tooling, claude-code, codex]
params:
  author: AI Beat Desk
  summary: >-
    Meta's new terminal coding agent reads CLAUDE.md and .codex/ skills as
    project context when its own AGENTS.md file doesn't exist. Any content
    in those files — internal URLs, API endpoint docs, project constraints
    — goes to Meta's servers on every call, and to Meta's training data if
    you're on the contributor tier.
---

Meta launched [Muse Code](https://techcrunch.com/2026/08/05/meta-launches-muse-code-an-ai-agent-for-large-code-bases/) on August 5th. It's a terminal-based coding agent built on Muse Spark 1.2, Meta's coding-focused model. The product is competent — persistent background agents, append-only crash-recovery logs, aggressive pricing — but one design choice is causing friction today: Muse Code reads `CLAUDE.md` and `.codex/` skill directories as fallback project context when `AGENTS.md` doesn't exist.

This means if your repository has a `CLAUDE.md` written for Claude Code, Muse Code picks it up automatically. Those project-level instructions — your internal API documentation, environment specifics, linting rules, whatever you put there — get sent to Meta's model API on every call.

## What actually goes where

The architecture is described as "local-first": the agent, subagents, and session logs run on your machine. Only the model inference is remote. But "local-first" doesn't mean your project context stays local. Every prompt includes the context files, and those go to Meta's servers as part of the API call.

The tier you're on determines what happens next:

- **Standard tier** ($1.25/$4.25 per million input/output tokens): your inputs are sent to Meta but not used for training.
- **Contributor tier** ($0.10/$0.20 per million tokens, roughly 12× cheaper): Meta uses your inputs and outputs to train future versions of Muse Spark.

The standard tier is the default. So by default, your CLAUDE.md contents travel to Meta's servers but are not retained for training. On contributor tier, they are.

## Why this is different from the usual cloud AI situation

Every cloud AI coding tool sends your code to the provider's API. That's not new or surprising. What's different here is the cross-tool context sharing.

When you write `CLAUDE.md`, you're writing it for Anthropic's system. The implicit trust model is: this context goes to Anthropic when I use Claude Code. You probably haven't thought about whether a competing tool will read the same file.

Muse Code's `AGENTS.md` compatibility is framed as a feature — "your existing Claude and Codex rules largely carry over, no rewriting needed." And it is convenient. But convenience and privacy sometimes cut in opposite directions. If your CLAUDE.md includes internal API base URLs, mentions staging environment specifics, or references non-public documentation, all of that is now going to Meta on every Muse Code session in that repository.

This is compounded if you're on the contributor tier without fully intending to be. The 12× price gap is real enough that teams may select it without thinking through what they're sharing. The codersera guide on Muse Code [puts it plainly](https://codersera.com/blog/muse-code-complete-guide-2026/): "once code is in the weights, no deletion request unwinds it. Block it at the policy layer rather than trusting a config string."

## What to do about it

If you're running Muse Code in repositories that have sensitive CLAUDE.md or AGENTS.md files:

- Review what's in those files before running Muse Code
- Use `muse init` to generate a project-specific `AGENTS.md` — once that file exists, CLAUDE.md isn't the fallback anymore
- Stay on the standard tier unless you've explicitly decided the training data tradeoff is acceptable
- Consider keeping internal API documentation, endpoint URLs, and environment specifics out of project-level context files entirely, since you can't always predict which tools will read them

The broader takeaway is that as coding agents proliferate and interoperate via shared config files, the trust perimeter around those files expands. CLAUDE.md, AGENTS.md, and equivalent files are becoming a new class of sensitive artifact — one that's easy to forget about because they look like documentation.
