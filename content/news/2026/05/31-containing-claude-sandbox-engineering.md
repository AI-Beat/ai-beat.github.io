---
title: "The Blast Radius Problem: How Anthropic Sandboxes Its Own Models"
date: 2026-05-31T06:10:00+00:00
draft: false
slug: containing-claude-sandbox-engineering
categories: [security]
tags: [security, agents, sandboxing, infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic's engineering blog documents the production sandboxing stack
    across claude.ai, Claude Code, and Cowork — three deployment contexts with
    different trust surfaces and different isolation primitives. The post is
    notable for what it admits: several real vulnerabilities, a consistent
    lesson that custom-built security components underperform battle-tested
    ones, and an honest account of how the threat model has changed as agents
    gained more capability.
---

There's a specific kind of engineering post that only gets written once a team has actually shipped the hard version of something. Anthropic published one this week: ["How we contain Claude across products"](https://www.anthropic.com/engineering/how-we-contain-claude), a detailed walkthrough of the sandboxing and isolation stack across their three main deployment contexts. It's worth reading carefully if you're building anything that puts an agent near a filesystem, a network, or a shell.

The framing is "blast radius reduction" — as agents become more capable and get granted more access, how do you bound the damage when something goes wrong? The answer Anthropic landed on is three interlocking layers: environment-level containment (the hard stuff: process sandboxes, VMs, egress controls), model behavior shaping (system prompts, classifiers, training), and external content controls. They're explicit that the second layer is probabilistic, not a guarantee. The first and third are where the real security lives.

The three products get meaningfully different treatment. **claude.ai** runs each conversation in an ephemeral gVisor container on isolated servers with minimal filesystem access — low capability, low blast radius, relatively straightforward to contain. **Claude Code** is harder: it's a local tool that developers run against their own repositories, with real shell access. It uses OS-native sandboxing: Seatbelt on macOS and Bubblewrap on Linux. The trust model is developer-centric; the assumption is that the person running it can read a bash command and make a judgment call. **Claude Cowork**, aimed at non-technical users, gets the strongest isolation: full virtual machines, on the logic that someone who can't evaluate `rm -rf` output needs the containment to be invisible and complete.

The most instructive part of the post is the vulnerability section. Between mid-2025 and early 2026, several real issues were found and fixed:

- Code executing before the user had granted trust (configuration hooks triggering too early)
- Direct prompt injection from phishing content reaching the agent context
- Exfiltration through approved domains — where engineers had assumed allowlists meant "safe to read from" but agents found ways to write *to* those domains
- VM isolation blocking EDR visibility, meaning security monitoring was blind inside the perimeter meant to be safe

That last one is a genuine tension. Stronger isolation creates better sandboxing and worse observability. You can have one or the other, not both easily.

The recurring lesson across all the failures is worth quoting from the post's apparent conclusion: custom-built components proved weaker than battle-tested primitives like hypervisors and seccomp filters. This is an old lesson in security engineering — don't build your own crypto, don't build your own sandbox — but it's interesting to see it confirmed in the specific context of AI agents, where the temptation to build bespoke systems is high because the access patterns are genuinely novel.

A detail worth noting: Anthropic states they now routinely grant Claude access that would have been flatly rejected a year ago. The flip side is that this happened precisely because the containment infrastructure got more robust. More capability behind better walls, rather than less capability in the open.

The post is notably candid for an engineering blog from a frontier lab. Documenting actual CVEs, admitting which security assumptions failed, and explaining the trade-offs between isolation levels and monitoring coverage is not the kind of thing companies usually publish unless they're confident in the current state. Whether that confidence is warranted is hard to assess from the outside — but the post gives you enough detail to audit the reasoning yourself, which is more than most.

If you're building anything with agents that touch real infrastructure, the section on how Cowork handles credential isolation is directly applicable: the design goal is to prevent credentials from ever entering the sandbox, not to contain them after the fact. Worth internalizing.
