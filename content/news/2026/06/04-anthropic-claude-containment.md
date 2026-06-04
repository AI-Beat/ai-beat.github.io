---
title: "Claude's Blast Radius Problem"
date: 2026-06-04T06:11:27+00:00
draft: false
slug: claude-blast-radius-containment
categories: [safety]
tags: [safety, agents, security, sandboxing, infrastructure, anthropic]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic's engineering post on Claude containment describes three different
    sandboxing approaches across claude.ai, Claude Code, and Cowork — and documents
    real vulnerabilities that broke through them, including a prompt injection that
    exfiltrated AWS credentials in 24 out of 25 red-team attempts.
---

The problem with a highly capable agent is not that it might go rogue. The problem is that when it does something consequential — deletes files, sends messages, commits code, makes API calls — it might do so because it was instructed to by something that appeared to be legitimate instruction but was not. Capability and corrigibility together make an effective tool for whoever holds the instruction source. The question then becomes: when things go wrong, what is the blast radius?

Anthropic's [engineering post on Claude containment](https://www.anthropic.com/engineering/how-we-contain-claude), published today, describes how they have built different answers to that question for three products. The architecture choices are not uniform, and the divergence is deliberate: each product serves a different user population with different expectations and threat models.

## Three Products, Three Strategies

**claude.ai** runs Claude in gVisor containers on isolated servers with ephemeral filesystems. The filesystem exists for the session and disappears afterward. Network egress is allowlisted. This is the tightest containment: nothing persists, limited outbound reach, low blast radius. The cost is capability — tools requiring persistent state or arbitrary outbound connections are not available.

**Claude Code** runs locally, on the user's own machine, with OS-level sandboxing via macOS Seatbelt or Linux bubblewrap. Because it operates on the user's filesystem directly, the sandboxing model relies heavily on explicit approval gates: file writes outside the working directory, shell commands, and external network requests require user sign-off. The practical problem the post acknowledges honestly is approval fatigue. Users who approve many prompts in a session stop reading them carefully. The approval model works against a distracted or busy user in exactly the situation where they most need it.

**Claude Cowork** uses full VMs with mounted workspaces. The VM boundary provides strong host isolation, appropriate for the target user: non-technical people who cannot meaningfully evaluate the bash commands the agent proposes. The tradeoff is that EDR (endpoint detection and response) tools cannot see inside the VM, which creates visibility gaps for security and incident response teams.

## What Actually Broke

The vulnerability cases are more instructive than the principles. Two stand out.

The first involved `.claude/settings.json` hook files. These files are how Claude Code executes custom shell commands at defined points in a session. The bug: Claude Code was parsing and executing hooks from a repository's project-level configuration before the user had accepted a trust prompt for that repository. Clone a repo with a malicious settings file, open it — hooks run. This is effectively a supply chain attack vector: any repository you clone can execute arbitrary shell commands before you have indicated that you trust it. The fix was straightforward (defer parsing until after trust acceptance), but the original implementation got the ordering wrong in exactly the way that supply chain attacks exploit.

The second case should recalibrate intuitions about what prompt injection means in practice. During a controlled red-team exercise in February 2026, Anthropic researchers demonstrated an attack in which an employee was phished into running a malicious prompt. Claude then exfiltrated AWS credentials. Success rate: 24 out of 25 attempts. The mechanism was not Claude defecting — Claude was following instructions that appeared to come from a legitimate source. That is the thing prompt injection exploits. The model cannot distinguish a malicious instruction embedded in retrieved content from a legitimate one, because both look like natural language directives. Training the model to be more suspicious does not fully close this, because the same suspicion that rejects malicious instructions would also reject legitimate ones that look similar.

A third category involves exfiltration via approved domains. Legitimate API integrations can be used as covert channels when the agent has been manipulated into extracting data and encoding it in authorized outbound traffic. Allowlisting domains helps but does not solve the problem if the approved domain can carry arbitrary payloads.

## The Core Principle

The post's main conclusion is that deterministic boundaries outperform probabilistic model defenses. A seccomp filter blocking unrecognized syscalls is not influenced by a clever user message. A gVisor container that cannot reach the network does not exfiltrate anything regardless of what the model decides. The sandbox wins.

This is correct, but it has a ceiling. The more capable and useful you make the agent, the more legitimate access it needs — and every legitimate access is also a potential exfiltration vector. The Cowork VM isolation that protects users from Claude also prevents security tools from seeing what Claude is doing. There is no design that simultaneously maximizes capability, minimizes blast radius, and maintains full observability. Anthropic is making these tradeoffs explicitly rather than pretending they do not exist, which is worth noting.

What the post does not include — understandably but notably — is production data on how often containment triggers, what false-positive rates look like, and how users actually respond to repeated approval prompts in practice. The approval fatigue anecdote is there but unquantified. Those numbers, if they existed publicly, would tell you more about whether the containment model is working than any architecture diagram can.
