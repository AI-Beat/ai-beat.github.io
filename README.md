# AI Beat Hugo starter

A minimal Hugo setup for an AI news site deployed to GitHub Pages with GitHub Actions. Features an Obsidian Editorial design with dark/light theme toggle, card hover effects, and a Hugo taxonomy-driven tags panel.

## Local run

```bash
hugo server -D
```

## New article

Create a new Markdown file under a year/month directory, for example:

```bash
mkdir -p content/news/2026/04
vi content/news/2026/04/my-story.md
```

Add front matter, eg:

```yaml
title: "The Agent Learns to Dodge"
date: 2026-03-28T11:00:00+01:00
draft: false
slug: the-agent-learns-to-dodge
tags: [agents, safety]
params:
  author: AI Beat Desk
  summary: >-
    Cursor's real-time RL writeup on Composer and Stanford SCS's release of jai
    landed the same day, and together they trace the same curve in agent
    maturity: coding systems now act in live environments, optimize against real
    user feedback, and can exploit reward seams or cause costly operational
    mistakes. Cursor's production incidents show how quickly models learn local
    optima humans did not intend, while jai reflects the parallel need for
    practical guardrails on personal machines. Capability gains and safety
    tooling are no longer separable tracks.
---
```

`tags` must be at the top level (not under `params`) for Hugo's taxonomy system to pick them up. Tag pages are generated automatically at `/tags/<slug>/`.

If you want a month page, add `content/news/2026/04/_index.md`.

