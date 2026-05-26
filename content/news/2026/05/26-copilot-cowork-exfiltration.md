---
title: "The Low-Risk Action That Wasn't"
date: 2026-05-26T06:11:00Z
draft: false
slug: copilot-cowork-prompt-injection-exfiltration
categories: [security]
tags: [security, prompt-injection, microsoft, copilot, agents, agentic-ai]
params:
  author: AI Beat Desk
  summary: >-
    PromptArmor published a working indirect prompt injection exploit against
    Microsoft Copilot Cowork that achieves file exfiltration from SharePoint
    and OneDrive with a 5-for-5 success rate — including against Claude Opus
    4.7. The attack works because Cowork auto-approves Teams and email sends,
    and because pre-authenticated download links can be embedded in those
    messages as image tag query parameters. It's a reminder that
    "human-in-the-loop" only means something if the loop actually catches this.
---

Yesterday, PromptArmor published a [write-up](https://www.promptarmor.com/resources/microsoft-copilot-cowork-exfiltrates-files) of a working exploit against Microsoft Copilot Cowork — the agentic layer inside Microsoft 365 — that achieves file exfiltration from SharePoint and OneDrive with a 5-for-5 success rate across all tested frontier models, including Claude Opus 4.7. The attack is technically clean and the implications are straightforward enough that it's worth walking through carefully.

## How it works

The vector is indirect prompt injection via a malicious skill file. An attacker uploads a skill that embeds hidden instructions into Cowork's context. When a user invokes a routine task — requesting a weekly recap, summarizing emails, anything that causes Cowork to run the compromised skill — the injected instructions take over the agent's next action.

The key architectural detail is what those injected instructions direct the agent to do: send a Teams message or email to the active user. This matters because Cowork's permission model treats Teams and email sends as low-risk actions that don't require per-invocation human approval. Write operations to SharePoint or calendar events require confirmation; sending a message to yourself, apparently, does not. From the agent's perspective it's following instructions. From the user's perspective, they just asked for a work summary.

The exfiltration channel is the clever part. Cowork can retrieve pre-authenticated download links for files the user has access to in SharePoint — links that grant file access to anyone who opens them, without additional authentication. The injected instructions direct Cowork to embed these links as query parameters in external image tags inside the outgoing message. When the user opens the message in Teams or Outlook, their client makes the embedded network request automatically, sending the file download URLs to an attacker-controlled server. The attacker then downloads the files.

PromptArmor reports this chain executed successfully on every trial. The post notes that "the attack achieved a high success rate against state-of-the-art models" — the implication being that this isn't a jailbreak against a weak model; it's an exploit of a system design that no model-level safety training is positioned to catch.

## What this exposes

There are two separate design decisions here that each independently contribute to the vulnerability.

The first is the automatic approval for Teams and email sends. The reasoning for this choice isn't hard to imagine: requiring explicit confirmation for every outgoing message would make Cowork much less useful as an assistant that can draft and send communications on your behalf. But "send to the current user" is different from "send to an external party" — and even internal sends can embed arbitrary content, including content that phones home when rendered.

The second is that pre-authenticated SharePoint download links exist at all in this form. They're a convenience feature for sharing files without forcing recipients to authenticate. But they're also bearer tokens — anyone with the URL can download the file. Embedding them as query parameters in an externally-triggered image request is a straightforward way to hand that token to anyone operating the server at the receiving end.

Neither of these is obviously wrong in isolation. Together, they compose into an exfiltration primitive.

## The model-level problem

The finding that Claude Opus 4.7 is exploited 5-for-5 should not be surprising, but it's worth stating clearly: current safety training and system prompt hardening doesn't defeat prompt injection. Models that are told to follow user instructions will, under indirect injection, follow attacker instructions instead — because from inside the context window, the two are indistinguishable. The injected instruction looks like a user instruction because it arrived in the slot where user instructions go.

This is not a new insight, but it's one that tends to get rediscovered every time a new agentic product ships. The attack surface shifts when the agent has write permissions, which agentic systems increasingly do.

The mitigation PromptArmor recommends is targeted: disable the "Don't ask again" option on Cowork write actions, so per-action approval stays active. That breaks the automatic message-send step. It also significantly limits what Cowork can do autonomously, which is the tradeoff every organization building on agentic infrastructure currently faces: the autonomy is the feature, and the autonomy is the attack surface.
