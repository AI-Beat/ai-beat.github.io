---
title: "The Plumbing Problem: Why Coding Agents Need Real VMs"
date: 2026-04-07T06:22:42Z
draft: false
slug: freestyle-agent-vms
tags: [agents, infrastructure, sandboxes]
categories: [infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    Freestyle launched today with <50ms VM forking for AI coding agent workloads,
    built on bare metal they own because cloud margins didn't pencil out.
    It's a signal that the agent infrastructure layer is serious enough
    to warrant serious systems work.
---

A startup called [Freestyle](https://www.freestyle.sh/) launched on Hacker News this morning with a technically credible claim: they can fork a running Linux VM—full memory state, live processes—in under 50 milliseconds.

That sounds like a systems flex, and it is, but there's a real problem underneath it.

Coding agents need isolation. When Claude Code or Cursor Agent or some bespoke workflow runs arbitrary code on your behalf, you want that code contained somewhere that won't bleed into anything else. Containers are the obvious answer—and for many workloads they're fine—but containers share a kernel, they don't give you real root, and they don't support nested virtualization or eBPF in the ways that some agent tasks need. Full VMs do. The catch is that full VMs are slow to boot and expensive to run at scale.

Freestyle's pitch is that they've solved the provisioning problem. Their VMs are full Debian systems with systemd, nested virtualization enabled, and multiple user accounts—the real thing—but they provision in under 700ms from API call to ready machine. More interestingly, the forking capability lets you snapshot a running VM and create a copy with the same memory state in under 50ms without pausing the original. For agent workloads that need to explore multiple branches of execution in parallel—think "run this test suite five different ways simultaneously"—that's a meaningful primitive.

They're also explicit about a business decision that reveals something about where the market is: they built on bare metal hardware they own rather than renting from AWS or GCP. The reason is cost. At the scale of "tens of thousands of agents" that they're targeting, cloud margins apparently don't work. Owning your own iron is a serious commitment and a sign that this isn't a weekend project.

The competitive space includes [E2B](https://e2b.dev/), Modal, Daytona, and Fly.io's Sprites product. The Freestyle founders argue their differentiator is power—full Linux rather than constrained sandboxes—and the forking capability, which they say is unique. The HN comments were reasonably skeptical but nobody seriously disputed the technical claims.

What's interesting about this launch isn't really the product features themselves—they're nascent, the pricing isn't public, and the use-case documentation acknowledges it needs work. What's interesting is what it signals: the agent infrastructure layer is mature enough that serious systems engineers are building dedicated primitives for it. A year ago, "sandbox for coding agents" meant running a Docker container. Now someone is building custom VM hypervisors and buying racks because the performance requirements are specific enough to warrant it.

The [HN thread](https://news.ycombinator.com/item?id=47663147) is worth reading. The founder (Ben) is responsive, the technical questions get real answers, and the comparison with competitors is relatively honest. The product is early but the engineering is not.

Agent sandboxing is boring infrastructure work, which means it's exactly the kind of thing that matters a lot once you're running anything serious in production. Freestyle is betting there's a real market for doing it well.
