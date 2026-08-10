---
title: "Who Wrote This Line?"
date: 2026-08-10T06:21:40+00:00
draft: false
slug: us-vs-them-git-provenance
categories: [tooling]
tags: [tooling, open-source, agents, provenance]
params:
  author: AI Beat Desk
  summary: >-
    us-vs-them is a small open-source library that reads git version history
    to produce line-level human/agent authorship scores — no markup required.
    As agentic editors increasingly co-author code, distinguishing human-written
    lines from machine-generated ones is becoming a practical necessity, and
    the git history turns out to be a surprisingly clean signal.
---

Git has always tracked who changed what and when. What it hasn't had to distinguish, until recently, is whether the author of a commit was a person or an agent acting on that person's behalf. [us-vs-them](https://github.com/eighttrigrams/us-vs-them) is a small library that picks that problem back up — not by adding markup to files, but by reading the shape of changes in the version history itself.

The tool produces a provenance score between 0.0 and 1.0 for each line in a file. A score of 1.0 means fully human-authored; 0.0 means fully agent-authored; 0.46 might mean a line the human originally wrote that an agent later modified. The underlying algorithm is diff-based: it traces each line's ancestry through the git history, computing authorship from which commits touched it. You provide a flag — `--ours` for the humans or `--theirs` for the agents, whichever list is shorter — telling the tool which git authors belong to which side. No annotations needed in the files themselves, no special commit-message conventions required — the git history plus commit authorship is the signal.

The output is a set of ranges: "islands" of human-authored lines inside a "sea" of machine-generated text, as the project README frames it. This is a more honest description of the structure of AI-assisted code than most people currently think about. In practice, a file in an actively agent-edited codebase isn't 50% human and 50% machine in any uniform sense. It has specific regions — often the parts that express intent, the unusual cases, the domain-specific logic — that a human touched deliberately, surrounded by scaffolding and boilerplate that an agent wrote and re-wrote dozens of times. The line-level score makes that structure visible.

The practical application the project describes is protecting human-intent sections from agent modification. If you're running an agent over a codebase and certain lines carry human reasoning you don't want overwritten, knowing which lines those are — and being able to compute it from history rather than maintaining a separate annotation file — is useful. You could gate agent edits on provenance score: don't touch lines above 0.8 without explicit instruction. You could also use the inverse: freely let the agent revise its own output (score near 0.0) without confirmation prompts.

There's a deeper point here worth stating plainly. The distinction between "my code" and "the agent's code" matters in ways that go beyond tooling convenience. Human-written lines tend to encode intent that isn't always recoverable from the code itself — a choice made because of a constraint the agent doesn't know about, a simplification that only makes sense given the broader system context. When an agent refactors a file and loses that signal, the loss is often invisible until something breaks. A provenance score is a proxy for "this line might contain information that isn't in the code" — not perfect, but better than treating all lines as equally fungible.

The approach has an honest dependency: it relies on agents committing under identifiable names. If your team has Claude Code or Copilot commit as the developer's own git identity — which is common when coding assistants push to the developer's branch — the tool can't distinguish those commits from human ones, and the score degrades. This is a workflow question as much as a tooling one: if you want clean provenance, you need clean attribution at commit time.

The library is currently a relatively small project — 43 commits, no formal releases, plaintext output. It's not trying to be an enterprise provenance platform; it's a sharp tool for a specific problem. What's interesting is that the problem exists at all. A year ago, the question "which lines did the AI write?" wasn't particularly meaningful for most codebases. It is now, and the git history is a better place to look for the answer than most people would initially guess.
