---
title: "Policy Outside the Agent's Reach"
date: 2026-10-12T06:12:00+00:00
draft: false
slug: microsoft-execution-containers
categories: [agents]
tags: [agents, safety, sandboxing, Windows, Microsoft, containment]
params:
  author: AI Beat Desk
  summary: >-
    Microsoft Execution Containers (MXC) reached general availability on October 7,
    bringing a cross-platform, policy-driven containment layer for AI agent
    workloads to Windows 11, macOS, and Linux — with isolation levels ranging
    from lightweight process containers up to experimental MicroVMs.
---

One of the structural problems with running AI agents is that the agent and its permissions live in the same process. If the agent decides — or is manipulated into deciding — to reach beyond its intended scope, there's often nothing between it and the filesystem, the network, or other processes except the code the developer remembered to add. That's a weak invariant.

[Microsoft Execution Containers (MXC)](https://blogs.windows.com/windowsdeveloper/2026/10/07/microsoft-execution-containers-policy-driven-containment-for-ai-agents/), which reached general availability on October 7, takes a different approach: move the policy outside the agent entirely. Developers declare what filesystem paths, network destinations, and process capabilities a workload needs in a JSON schema. MXC enforces that boundary at the OS level. The agent can't grant itself more access because the policy isn't in its control plane.

The architecture has four containment modes, matched to different risk levels. Process containers run on Windows 11, macOS, and Linux — lightweight, low overhead, suited to typical coding or research agents. Session containers (Windows 11 only) give each agent an isolated OS session with its own identity, appropriate when an agent needs a GUI or a persistent environment. WSL containers run a Linux toolchain inside Windows with standard WSL isolation. MicroVMs, still experimental, provide hardware-backed virtualization for workloads where you genuinely want a hard boundary — think agents executing arbitrary code from external inputs.

The policy model has three enforcement modes: enforce (blocks ungranted operations), learning (blocks and logs), and permissive (logs only). The learning mode is useful during development to discover what a new agent actually needs without having to enumerate it upfront. Activity reports come out as JSON, which means they're inspectable and diffable — reasonable for audit workflows.

Cross-platform support via a unified SDK is the most practical detail here. An agent workload containerized under MXC can run on a developer's Mac, a Linux CI node, or a Windows Server environment with the same policy file. That removes the "this only works in production" problem that often plagues sandboxing tools.

What MXC doesn't solve is anything at the semantic layer. If an agent has legitimate write access to a directory, it can still write malicious content there. If it has network access to an API endpoint, it can exfiltrate data through that endpoint. Policy-driven containment is about enforcing declared scope, not about understanding intent. That's a meaningful but narrow guarantee — closer to a capability system than to a firewall with deep packet inspection.

The more interesting downstream question is whether policy-as-data makes it practical to audit and version agent permissions the way you'd audit and version application permissions. If a production agent's policy file lives in version control alongside its code, you get a machine-readable record of what it was allowed to do at each deployment. That's a modest improvement over the current state of affairs, where agent permissions are usually implicit in whatever API keys were handed to the process.

GitHub Copilot, OpenAI Codex, LM Studio, and Unsloth already support MXC. Anthropic Claude Code is listed as forthcoming. The SDK and schema are open, so third-party agent frameworks can add support without waiting on Microsoft.
