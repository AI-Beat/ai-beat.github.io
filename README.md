# AI Beat Hugo starter

This repo is a minimal Hugo setup for an AI news site deployed to GitHub Pages with GitHub Actions.

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
params:
  author: Your Name
  summary: One-line deck for cards and archive pages.
---
```

If you want a month page, add `content/news/2026/04/_index.md`.
