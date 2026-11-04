---
title: "LLM 0.37 and the AI-Reviewed AI PR"
date: 2026-11-04T06:12:30+00:00
draft: false
slug: llm-tool-0-37
categories: [tools]
tags: [open-source, llm-tools, embeddings, openai-compatible]
params:
  author: AI Beat Desk
  summary: >-
    Simon Willison's LLM CLI hit 0.37 on November 2, adding gpt-6.1 models,
    a simpler OpenAI-compatible endpoint setup command, and multimodal embeddings.
    The more interesting note is that nearly all 12 contributing authors used AI
    assistance, and Willison uses Opus 5.5 or GPT-6 Astra to triage those
    submissions — a small but clear snapshot of where open source maintenance is now.
---

[LLM 0.37](https://simonwillison.net/2026/Nov/2/llm/) landed on November 2, bundling about two months of changes to Simon Willison's widely-used command-line tool and Python library for running language models.

The functional additions are straightforward. The release adds `gpt-6.1-sol` and `gpt-6.1-terra`. A new `llm openai endpoint add` command makes it easier to configure additional models that expose an OpenAI-compatible API — previously this involved more manual YAML editing. The default model for users who haven't set their own changed from `gpt-5.6-luna` to `gpt-6-luna`, which Willison notes costs about half as much. Embeddings now accept binary content with explicit content types, which matters for multimodal models like Gemini Embedding 2 that can embed text, audio, images, and video in the same space.

The more interesting note is in the contributions section. The release includes fixes from 12 authors, and Willison describes nearly all of them as AI-assisted. His description of handling them is worth quoting: they're "bug reports with attached examples." The AI component drafts a fix along with the reproduction case, which means the reviewer can often verify the problem immediately rather than trying to reproduce it from scratch. Sometimes he merges the contribution; sometimes he writes his own fix using the report as a spec.

Willison says he uses Opus 5.5 or GPT-6 Astra to help triage these submissions — scanning them, surfacing the most useful ones, and flagging anything that looks problematic. The loop here is complete: users run AI to write PRs, the maintainer runs AI to review them, and the tool being maintained is itself an AI interface.

This is a small signal, but it's representative of where a lot of active open source is right now. The concern a year ago was that AI-generated PRs would flood maintainers with low-quality submissions that looked plausible but weren't. Willison's experience is more measured: he's not finding it to be a significant problem, in part because the AI-generated PRs tend to come with evidence. A bug report that includes a minimal reproduction and a proposed fix is easier to evaluate than one that doesn't, regardless of whether a person or a model wrote it.

Whether this scales to larger projects with more complicated codebases and less clearly-scoped issues is an open question. But for a focused utility like LLM — which has a clear scope, well-defined interfaces, and a maintainer who knows every part of the codebase — the current equilibrium seems to be working.
