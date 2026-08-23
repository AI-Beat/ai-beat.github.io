---
title: "MCP Gets Serious About Production"
date: 2026-08-23T06:09:24Z
draft: false
slug: mcp-roadmap-2026
categories: [agents]
tags: [mcp, agents, protocols, security]
params:
  author: AI Beat Desk
  summary: >-
    The new MCP roadmap moves the protocol from an experiment in tool-calling
    toward something that can survive enterprise procurement: agent identity
    via DPoP and Workload Identity Federation, async Tasks heading into the
    core spec, and progressive tool discovery to keep large catalogs from
    overwhelming model attention.
---

The [Model Context Protocol published its 2026 roadmap](https://blog.modelcontextprotocol.io/posts/mcp-roadmap/) on Friday, and reading it alongside the July 28 specification release gives a clearer picture of where the protocol is actually headed. The short version: MCP is trying to shed its reputation as an interesting prototype and become something you'd stake a production system on.

The five priority areas span a range of abstraction levels, but a few stand out.

**Tasks toward the core spec.** The Tasks extension (SEP-2663) is the roadmap's most load-bearing item. Tasks provide the async primitive MCP currently lacks: a client initiates work, retrieves results later, and the server can push events mid-flight. That sounds like a minor convenience until you realize it's the only clean way to handle a multi-step agent workflow that runs longer than a single HTTP request. Without Tasks, everything gets shoehorned into synchronous calls or bolted on outside the protocol. The roadmap doesn't give a ship date, but naming a SEP number and saying it needs to "compose cleanly with Triggers" signals active spec work rather than aspirational notes.

**Agent identity without API keys.** The current state of MCP auth is awkward: servers issue long-lived tokens, clients hand them over, and nobody can tell which *agent* is calling versus which human issued the credential. The roadmap addresses this with two mechanisms. [Demonstrating Proof of Possession (DPoP)](https://datatracker.ietf.org/doc/html/rfc9449) binds a token to a specific key pair so stolen tokens can't be replayed from a different client. Workload Identity Federation with the ID-JAG grant goes further: it lets agents obtain short-lived credentials via an organizational identity provider, the same way cloud workloads get IAM credentials today. Together these move MCP toward the model where an agent has a cryptographic identity rather than a shared secret, which is the prerequisite for meaningful audit logs and least-privilege policies.

**Progressive tool discovery.** This one is underrated. The current model dumps the full tool catalog into the context at connection time. For small servers that's fine; for a server with hundreds of tools it's actively harmful — model performance on tool selection degrades as the candidate list grows. The roadmap proposes letting servers expose a minimal initial surface and reveal more tools as the conversation narrows. This is the right architectural move and probably overdue: it mirrors how humans search documentation rather than memorizing it.

**HTTP-native for local servers.** The July 28 release standardized HTTP as the transport for remote servers. The new roadmap extends that unification to local servers, replacing stdio-over-HTTP patchwork with proper Streamable HTTP. This mostly matters for tooling — debuggers, conformance tests, and proxies that currently need different code paths for local versus remote deployments.

What the roadmap doesn't address is equally telling. There's no mention of versioning strategy for servers that need to stay compatible across spec releases, and nothing about discovery — how a client finds what servers exist is still left to ecosystem convention. These gaps will matter as the number of production MCP servers grows. For now the protocol team seems to be deliberately sequencing: get the async and identity primitives right first, then tackle the directory problem.

The SEP process (Specification Enhancement Proposals) is also getting explicit priority treatment — proposals aligned with the five roadmap areas get expedited review. That's a small governance detail that actually matters if you're deciding whether to invest in writing a formal proposal versus just shipping a workaround.

MCP sits in an interesting position right now. It's become the de-facto standard for agent tool integration almost by default, which means its rough edges are everyone's rough edges. A roadmap that takes security and async semantics seriously is exactly what the protocol needs to stay in that position rather than getting supplanted by something built specifically for production from the start.
