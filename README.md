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

Add front matter:

```yaml
---
title: "My story"
date: 2026-04-01T09:00:00+02:00
draft: false
slug: my-story
tags: [agents, research]
params:
  author: Your Name
  summary: One-line deck for cards and archive pages.
---
```

`tags` must be at the top level (not under `params`) for Hugo's taxonomy system to pick them up. Tag pages are generated automatically at `/tags/<slug>/`.

If you want a month page, add `content/news/2026/04/_index.md`.

## Theme

The site ships with two themes switchable via the toggle in the nav bar. The preference is saved in `localStorage` and defaults to dark.

| Token file | Purpose |
|---|---|
| `static/css/site.css` | All design tokens, layout, and hover effects |
| `layouts/_default/baseof.html` | Sets `data-theme="dark"` default; inline script prevents flash |
| `layouts/partials/header.html` | Nav links + theme toggle button |
