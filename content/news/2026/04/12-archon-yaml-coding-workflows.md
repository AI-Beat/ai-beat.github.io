---
title: "Giving AI Coding Agents a Script to Follow"
date: 2026-04-12T17:34:47Z
draft: false
slug: archon-yaml-coding-workflows
categories: [tooling]
tags: [tooling, agents, developer-tools]
params:
  author: AI Beat Desk
  summary: >-
    Archon wraps AI coding agents in versioned YAML workflows — DAG pipelines
    with Prompt, Bash, Loop, and Approval nodes — and runs each task in an
    isolated git worktree. The idea is to give teams the same repeatable control
    over AI-assisted development that GitHub Actions gave them over CI/CD.
---

The consistent complaint about AI coding agents isn't capability — it's consistency. Ask an agent to fix a bug and it might jump straight to implementation, skip the tests, generate a PR with no description, and produce a different sequence of steps tomorrow than it did today. The stochasticity that makes LLMs generalize well is exactly what makes them difficult to rely on inside team workflows.

[Archon](https://github.com/coleam00/archon), an open-source project maintained by coleam00 and Wirasm, takes a CI/CD-style approach to this problem: encode your development process once, in YAML, and the agent follows that script every time. Version 0.3.6 shipped today.

The core abstraction is a workflow file that describes a DAG — a directed acyclic graph — of typed nodes. A **Prompt node** sends instructions to whichever AI assistant is configured and captures the output as a named variable. A **Bash node** runs a deterministic shell command. A **Loop node** iterates until a completion condition is met. An **Approval node** pauses execution and waits for a human to sign off. Outputs from any node can be passed downstream with a `$nodeId.output` substitution syntax, so a planning step can feed its output directly into the implementation prompt, which feeds into the validation command. Seventeen built-in workflow templates cover the common patterns: fix a GitHub issue, implement a feature, review a PR, perform a refactor.

The other interesting design choice is isolation. Each workflow run gets its own [git worktree](https://git-scm.com/docs/git-worktree), synchronized from origin before execution. This means you can kick off five parallel fixes without any branch conflicts — each agent operates on its own checked-out copy of the repo, cleans up when the PR is merged, and leaves no stale state. For teams running background agents triggered by GitHub webhooks or Slack messages (both supported), this matters: you can queue tasks without serializing them.

What this looks like in practice is something closer to how teams already think about CI/CD pipelines than how they've been thinking about AI coding. A GitHub Actions workflow that runs lint, then tests, then builds, then deploys is not a new idea — you encode your process once and trust it to execute consistently. Archon applies the same pattern to a workflow that involves LLM calls. The AI-powered steps (planning, code generation, PR description) are nodes in the same graph as deterministic steps (running tests, checking formatting). The graph structure makes the dependencies explicit and the execution reproducible.

There are real limitations worth naming. YAML workflow definitions front-load the work of capturing your team's process — if you haven't thought carefully about how features should be built, a YAML file won't help you figure that out. And the "determinism" here is structural, not semantic: the same prompt sent to the same model will still produce different outputs on different runs. What you get is consistent *structure* around the AI calls, not consistent *results* from them. The Loop node is specifically designed to handle the case where the first attempt at implementation doesn't pass tests, letting the agent retry with feedback — but that's iteration management, not guaranteed convergence.

v0.3.6, released this morning, is largely a UI pass: artifact file paths in chat messages are now clickable, workflow execution views show status, duration, node counts, and produced artifacts, and loop node iteration progress is more visible. The backend gets a fix for environment variable leakage in multi-tenant deployments and better handling of first-event timeouts. Not a major architectural shift — steady progress on a tool sitting at 16,000+ GitHub stars, which suggests the workflow-as-code framing is resonating with teams already running agents in production.
