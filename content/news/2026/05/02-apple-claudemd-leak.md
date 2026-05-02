---
title: "Apple Shipped Its Claude Code Config to Production"
date: 2026-05-02T06:16:07+00:00
draft: false
slug: apple-claudemd-leak
categories: [tooling]
tags: [apple, claude-code, anthropic, tooling, build-process]
params:
  author: AI Beat Desk
  summary: >-
    Apple Support app v5.13 accidentally shipped two CLAUDE.md instruction files
    in the app bundle, exposing internal architecture context including a shared
    UI library called SAComponents and a chat module with three participant roles.
    Apple pushed v5.13.1 hours later to remove them, but not before the contents
    circulated.
---

Yesterday, Apple shipped a routine update to the Apple Support app — version 5.13 — and forgot to strip its CLAUDE.md files from the bundle before release. Developer Aaron Perris [spotted them and posted screenshots](https://news.ycombinator.com/item?id=47973378), the contents circulated quickly, and Apple pushed a fix hours later.

CLAUDE.md is the per-project instruction file that [Claude Code](https://claude.ai/code) reads at the start of every session — the rough equivalent of a `.cursor/rules` or a project-level README aimed at an AI assistant rather than a human. It typically holds architectural context, naming conventions, preferred libraries, and anything else you'd otherwise have to re-explain every session. Developers keep these in source repos; they are not meant to ship to end users.

Two files were in the bundle. One documented `SAComponents`, an internal-only shared UI library [used across Apple platforms including visionOS](https://tech.yahoo.com/ai/claude/articles/apple-using-claude-inside-company-114500152.html). The other described the app's chat module — three participant roles (client, agent, assistant) with conditional compilation flags. The files contained architecture notes and coding conventions, the kind of dense contextual detail that makes a CLAUDE.md useful to Claude Code and that you'd never expect to see in a shipping app.

None of the exposed information is a security catastrophe. Internal library names and message routing conventions are not secrets at the level of private keys or signing certificates. But the incident is worth noting for a couple of reasons.

First, it is concrete evidence that Apple engineers use Claude Code — Anthropic's third-party coding assistant — when building production consumer apps, not just experimental tooling or internal prototypes. The files were present because someone on the team was actively working with Claude Code on the Support app codebase, and the CLAUDE.md ended up in the build artifact because something in the packaging pipeline didn't exclude it. Apple has its own AI platform (Apple Intelligence, on-device models in iOS 18+), and it is still reaching for an external coding assistant to get work done faster.

Second, it is a reminder that CLAUDE.md files carry non-trivial context about system architecture. A file that's useful precisely because it's detailed is also a file that you don't want shipping to the public. This is a build tooling problem — CLAUDE.md, `.cursor/rules`, Copilot workspace configs, and their equivalents should probably be on a default exclusion list alongside `.env` and `.git`. Most projects don't have that wired up yet.

The actual fix is simple: add `CLAUDE.md` to your `.gitattributes` export-ignore or to whatever packaging step controls what ends up in the release artifact. Apple's version of that step was evidently missing it. The emergency patch suggests they found and fixed it quickly once the public caught on — the social cost of shipping a CLAUDE.md is low, but it's the kind of thing that prompts a review of what else might be leaking out of the build.

Apple isn't unusual here. The number of large companies quietly embedding Claude Code and similar tools into their developer workflows is growing fast, and most of the resulting configuration lives in files that weren't designed with accidental exposure in mind. Apple just happened to be the one that got caught.
