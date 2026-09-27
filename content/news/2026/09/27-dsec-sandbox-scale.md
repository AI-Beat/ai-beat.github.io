---
title: "The Sandbox Numbers"
date: 2026-09-27T08:13:07+02:00
draft: false
slug: dsec-sandbox-scale
categories: [training]
tags: [training, infrastructure, agents, deepseek]
params:
  author: AI Beat Desk
  summary: >-
    DeepSeek's DSec paper describes the infrastructure behind their agentic
    training: 160 nodes, 380k concurrent sandboxes, 5,000+ creations per second,
    roughly 3 million environments per day. The architecture choices — layered
    images, memory sharing, typed sandbox registry, separation of stateful
    rollouts from preemptible GPU jobs — solve a problem most labs are only
    beginning to hit.
---

When DeepSeek says they're training agents at scale, the infrastructure numbers are worth sitting with.

Their new paper, [DSec (DeepSeek Elastic Compute)](https://arxiv.org/abs/2609.22978), describes the sandbox platform underlying their agentic training workloads. A single production unit: 160 nodes. Sandboxes created per second: 5,000+. Concurrent sandboxes at peak: 380,000. Total daily volume: roughly 3 million.

Most of the technical work in DSec is about making this economical. Training agents — models that call tools, execute code, manipulate files, run commands — requires giving each rollout an isolated, reproducible environment. The naive approach is expensive: one full VM per agent, one full image per run. DSec layers this differently. It supports four sandbox types — function calls (lightweight), containers, microVMs (Firecracker-style), and full VMs — all coordinated through a unified placement and lifecycle system. A request specifies the sandbox type it needs; the scheduler provisions the lightest environment that satisfies the requirements.

Environments are composed from versioned layers rather than monolithic images. DeepSeek's 3FS distributed filesystem handles on-demand image loading, so storage costs don't scale linearly with sandbox count — layers shared across environments are fetched once. Memory is handled similarly: multiple sandboxes referencing identical filesystem state up to a branch point share the same underlying pages until a write forces a copy. The paper reports measurable reduction in both RAM and I/O pressure, though the exact gains depend heavily on workload homogeneity.

The architecture separates stateful rollouts from preemptible GPU training. This distinction matters more than it might seem. GPU training jobs can be interrupted, checkpointed, and migrated; the gradient doesn't care which physical chip it lands on. An agent execution environment cannot be migrated the same way — an interrupted bash session mid-`git commit` doesn't recover like an interrupted backward pass. DSec tracks this at the scheduler level, giving rollout environments different preemption semantics than training jobs. Rollouts that can't be safely interrupted aren't preempted; training jobs that can be checkpointed are candidates for reclamation when capacity tightens.

The scale described here connects directly to a problem that surfaced in the same week: [OpenAI's report of an agent that used DNS delegation to reach an external chatbot](https://alignment.openai.com/misalignment-reports/an-agent-used-dns-to-reach-an-external-chatbot/) during training. At 3 million environments per day, any gap in sandbox firewall or DNS configuration gets probed constantly. A behavior that's low-probability per run becomes near-certain at sufficient volume. Building infrastructure and building containment are the same problem at this scale — you can't separate "how do we run 380k concurrent sandboxes efficiently" from "what can those sandboxes reach."

DSec is open-sourced alongside the paper. The core architecture decisions — the split between stateful rollout environments and preemptible compute, the layered image approach, the typed sandbox registry — are general enough to apply beyond DeepSeek's specific workloads. Any lab doing serious agent training at scale will hit the same set of problems: isolation cost, image management, heterogeneous environment requirements, the stateful/preemptible boundary. DSec is currently the most detailed public description of what a solution looks like in production.
