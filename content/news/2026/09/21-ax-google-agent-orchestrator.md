---
title: "Google Ships kubectl for Agents"
date: 2026-09-21T06:11:26+00:00
draft: false
slug: ax-google-agent-orchestrator
categories: [agents]
tags: [agents, infrastructure, google, orchestration, open-source, kubernetes]
params:
  author: AI Beat Desk
  summary: >-
    Google open-sourced AX, a declarative orchestration platform for running
    agent workloads at scale. The Kubernetes-familiar API — apply, get, describe,
    watch — is deliberate: they want infrastructure engineers, not AI researchers,
    to be the ones running this. The interesting claim is that agents need a
    different resource model than containers, specifically around idle cost and
    untrusted code isolation.
---

Google [shipped v0.3.0 of AX (Agent Executor)](https://github.com/google/ax) this morning, the open-source orchestration layer for agent workloads they've been building since May. The tagline — "run billions of autonomous agent workloads in a cluster" — is ambitious enough to dismiss, but the technical design makes a specific argument worth examining.

The system defines four resource types, expressed as Kubernetes-style `ax.io/v1alpha1` manifests:

- **Task** — the execution unit: sandboxed, resource-bounded, runs untrusted agent code
- **Workspace** — environment setup: pre-wires Git repos, MCP servers, and skill packages so tasks start warm rather than spending time on setup
- **Gateway** — network policy: restricts which hosts an agent can reach
- **Model** — LLM configuration: centralizes which model a cluster of agents calls, with credentials pulled from Kubernetes secrets

The CLI mirrors kubectl: `ax apply`, `ax get`, `ax describe`, `ax watch`, `ax delete`, plus `ax suspend`, `ax resume`, and `ax ssh` for debugging. If you've run anything on Kubernetes, you can read AX configs without a tutorial. This is clearly intentional — Google wants the infrastructure engineers who already run production clusters to be able to pick this up without retraining.

The interesting design claim is about idle economics. The README notes that agents spend most of their wall time waiting — on API responses, on human feedback, on external services — and that standard container orchestration wastes those cycles because containers hold their resource allocation while idle. AX's answer is that agents should be suspendable: checkpointed to storage and resumed in under a second with "zero cold-start delay." The stated goal is dense multiplexing — only consume CPU and memory when actually computing. Whether that claim survives real workloads is TBD, but the framing is right: the cost model for agent fleets is different from the cost model for web services, and infrastructure designed for the latter will be wasteful for the former.

The security story is also specific. Agent tasks, by definition, run code the orchestrator doesn't fully control — instructions flow from an LLM whose outputs aren't auditable at deploy time. AX treats this directly: Tasks run in isolated sandboxes with explicit CPU and memory limits, Gateways enforce outbound network allowlists at the infrastructure level rather than relying on the agent's own code to behave, and Workspaces define exactly what a task can see. This is more opinionated than "run it in a container and hope for the best," which is the current default for most agent deployments.

The warning in the docs that "major breaking changes" are likely before a stable release should be taken at face value — this is pre-alpha infrastructure, not something to build a production stack on today. But it's worth reading the design now because Google is essentially publishing a theory of what agent infrastructure needs to be: a resource model that separates idle agents from active ones, isolation strong enough to run untrusted outputs, and an API familiar enough that the existing infrastructure engineering community can run it.

The question this raises is whether agent orchestration turns out to be a platform problem — where Google's Kubernetes heritage and cluster-at-scale experience give them durable advantage — or a tooling problem, where the right abstractions are simple enough that an open-source project or a startup can match the incumbent. Kubernetes won the container orchestration war in part because Google seeded it with the experience of running Borg internally. AX is a smaller bet, but the move is recognizable: ship the infrastructure layer as open source, establish the API conventions, and be the company that knows how to run it in production.

Whether agent workloads ever actually reach the scale that justifies a dedicated orchestration layer is still unclear. But if they do, Google just put their design on the table.
