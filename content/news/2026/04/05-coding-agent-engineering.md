---
title: "The Harness Is the Product"
date: 2026-04-05T08:30:00+02:00
draft: false
slug: coding-agent-engineering
tags: [agents, engineering, tools]
categories: [tooling]
params:
  author: AI Beat Desk
  summary: >-
    Sebastian Raschka published a technical breakdown of what a coding agent
    harness actually needs — six components that often matter more than the model
    itself. The same day, Imbue's case study on running 100+ Claude agents in
    parallel to test and improve their own tooling arrived on Hacker News. Together
    they sketch what production-grade agent engineering looks like right now.
---

Two pieces on the engineering side of coding agents landed within a day of each other, and they complement each other in a useful way.

Sebastian Raschka published ["Components of a Coding Agent"](https://magazine.sebastianraschka.com/p/components-of-a-coding-agent) on April 4. It's a careful breakdown of the six things a good agent harness needs to get right. Brief summary:

**Live repo context**: The agent needs to understand its environment before taking action — git branch, project structure, relevant documentation gathered upfront rather than discovered mid-task. **Prompt shape and cache reuse**: Rebuild as little as possible each turn. A stable prefix containing instructions and tool descriptions gets cached; only the changing elements (conversation history, new requests) get appended. Prompt cache hits are a meaningful cost and latency lever. **Tool access with validation**: Tools with clear boundaries, validated programmatically before execution — not just suggestions embedded in prose. **Context reduction**: Long outputs get clipped, repeated file reads get deduplicated, older entries compress more aggressively than recent ones. **Structured session memory**: Full transcript plus a separate working memory of distilled current-task state, enabling session resumption. **Bounded subagents**: Parallel subtasks delegated to agents that inherit enough context to be useful but operate under tighter constraints than the main agent.

The closing observation is the one that sticks: "The harness can often be the distinguishing factor that makes one LLM work better than another." The implication is that swapping models while keeping a weak harness will underperform keeping the model and upgrading the scaffolding. That matches most practitioner experience but rarely gets stated this directly.

Then there's [Imbue's case study](https://imbue.com/product/mngr_part_2/) on running their open-source agent orchestration tool, mngr, to improve itself. The setup is neat: `tutorial.sh` contains blocks of demo commands. For each block, a coding agent derives one or more pytest functions — more exhaustive than the original commands, covering happy and unhappy paths. Then mngr spawns one agent per test, each with its own environment, to run, debug, fix, and improve its assigned test. The outputs get merged back in. The whole pipeline runs on 100+ parallel agents.

A few things about this case study stand out. First, the self-improvement angle: using agents to maintain the tests for the tool you use to run agents. It's turtles, and it apparently works. Second, the comment thread on Hacker News surfaced the real engineering constraints Imbue didn't fully discuss. Token budget economics are non-trivial at scale: each agent processing a codebase burns 20–50K tokens on context setup before doing any actual work. A hundred agents running hourly accumulates fast. Third, observability: with a single agent, you read logs. With a hundred, you need pattern detection across failure distributions. Silent coordinated timeouts look different from model rate limits look different from genuine test failures. The hard part isn't concurrency — it's knowing what's wrong when things break.

Both pieces are pointing at the same thing: model capability is increasingly table stakes. The differentiation is in the surrounding system — the harness, the context management, the orchestration, the tooling for understanding what 100 agents actually did. This is not a new observation but it's one that keeps getting validated by people building real things.

Raschka's breakdown is a useful checklist for anyone evaluating or building an agent harness. Imbue's case study is a useful data point on what the friction looks like in practice when you take that checklist seriously and run it at scale.
