---
title: "The Unix-Friendly LLM CLI Grows Up"
date: 2026-08-09T06:10:00+00:00
draft: false
slug: llm-0-32-reasoning-traces
categories: [tools]
tags: [llm, cli, python, tooling, reasoning, simon-willison]
params:
  author: AI Beat Desk
  summary: >-
    LLM 0.32, Simon Willison's CLI tool for talking to hundreds of models,
    ships its most significant update since launch: reasoning traces go to
    stderr, server-side tools replace local execution, and conversation logs
    adopt a Git-style content-addressable format. Three changes that each
    solve a real design problem cleanly.
---

[LLM 0.32](https://simonwillison.net/2026/Aug/4/new-release-of-llm/) dropped on August 4th. It's the most architecturally interesting update Simon Willison's CLI tool has had in a while, and it's worth unpacking why a few of the choices here are the right answer to genuinely tricky problems.

If you haven't used `llm`, the short version: it's a Python CLI and library that lets you talk to OpenAI, Anthropic, Gemini, Ollama, and a hundred others through a consistent interface, with plugin-based model support, conversation logging, and composable Unix-style piping. It's the tool for people who want to wire AI into shell scripts and data pipelines without depending on whatever chat UI the week has produced.

## Reasoning to stderr

The most immediately visible change is that reasoning models now show their thinking — but they write it to stderr, not stdout.

This is exactly right. If you've ever tried to pipe a reasoning model's output to `jq` or `grep` and found yourself fishing chunks of `<think>` tags out of the stream, you know the problem. Stdout is for data; stderr is for information the human is watching. The split maps cleanly to how Unix tools work: `llm "explain this" 2>/dev/null | wc -w` gets you the word count of the answer, not the thinking. `llm "explain this" 2>&1 | less` lets you page through everything if you want.

The `-R` / `--hide-reasoning` flag drops it entirely when you don't want even the stderr noise. Clean.

## Server-side tools

The bigger architectural shift is how tool calling works. LLM now supports provider-hosted tools — OpenAI's CodeInterpreter and WebSearch, Anthropic's web fetch, code execution, and MCP connector — rather than only locally-executed function tools.

This matters because the tool-execution model has quietly split. A locally-defined tool means the model calls it, your client runs the code, and you manage the round-trip. A server-side tool means the provider's infrastructure handles execution — you don't write the tool handler, you just declare you want it available. For common things like "search the web" or "run this snippet," delegating to provider infrastructure is simpler. The provider has already done the sandboxing, rate-limiting, and format handling.

LLM's new `stream_events()` method in the Python API makes the distinction explicit: you get typed events for reasoning text, output text, tool calls, and attachments. Your code can route each event type differently rather than parsing a flat text stream.

## Content-addressable conversation logs

The logging system got a Git-inspired redesign. Previously, appending a message to a conversation meant storing the full message history in JSON again — if you had a 50-message conversation, the 50th entry stored all 50 messages in full. The new system stores each unique message content once and references it by hash, same as how Git objects work. The `llm logs` command still reconstructs the original format transparently.

The practical win: conversation logs don't balloon. The implementation detail worth noting: this is a real backwards-compatible migration, not just a format change. The `llm logs` command handles both formats and presents a unified view.

## What this adds up to

Each of these three changes solves a specific design problem that reasoning models and tool-calling created. Reasoning traces needed to be visible but not contaminate output streams. Server-side tool execution needed to be first-class, not a workaround. Conversation state needed a storage model that didn't duplicate data on every turn.

None of this is revolutionary — these are all known patterns applied to a specific domain. But that's often what good tool-building looks like: recognizing which existing ideas (Unix file descriptors, Git content-addressing) fit a new problem well, rather than inventing something new. LLM 0.32 lands in a good place.

Install or update with `pip install -U llm` or `uv tool install llm`.
