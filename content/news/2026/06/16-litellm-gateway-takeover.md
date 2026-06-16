---
title: "The Gateway Was the Weak Link"
date: 2026-06-16T06:10:00+00:00
draft: false
slug: litellm-gateway-takeover
categories: [security]
tags: [security, infrastructure, agents, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Obsidian Security chained three bugs in LiteLLM, the open-source proxy
    that sits in front of more than 100 model providers, to turn a default
    low-privilege account into full admin and remote code execution. The
    interesting part isn't the CVSS 9.9 — it's that a compromised gateway can
    rewrite LLM responses in flight and forge tool calls into agents like
    Claude Code, which makes the proxy itself part of the attack surface
    agent builders need to model.
---

LiteLLM has become one of those pieces of infrastructure that quietly ended up underneath everything. It's the proxy a lot of teams put in front of OpenAI, Anthropic, Gemini, and dozens of other providers so their application code can call one unified API and not care which model is on the other end. That's exactly the kind of component you want to be boring and trustworthy, because everything downstream — every prompt, every credential, every tool call — flows through it.

[Obsidian Security's writeup](https://www.obsidiansecurity.com/blog/litellm-privilege-escalation-rce) shows what happens when it isn't. They found three separate bugs that chain into a complete takeover, starting from a default `internal_user` account with no special privileges at all.

The first hole (CVE-2026-47101) is in key generation. When a user calls `/key/generate`, LiteLLM lets them specify an `allowed_routes` field describing which API routes the new virtual key can hit — and stores whatever the caller sends without checking it against what that user is actually allowed to access. Set `allowed_routes: ["/*"]` and you've minted yourself a key with a wildcard, including admin-only endpoints.

The second (CVE-2026-47102) is a missing field-level check on `/user/update` and `/user/bulk_update`. The route itself doesn't require admin rights to call, and the handler doesn't stop a user from including their own `user_role` in the update payload. So a regular user can just set their role to `proxy_admin` and the system obliges.

With admin access in hand, the third bug (CVE-2026-40217) turns privilege into code execution. LiteLLM's custom-guardrails feature lets admins upload Python that runs against requests via `exec()`, with `__builtins__` left available — meaning the "guardrail" is really an unrestricted code execution endpoint once you're allowed near it. A test endpoint for guardrails had a second, independent route to the same place, protected only by a regex deny-list that doesn't survive contact with someone trying to defeat it.

Stack the three together and a low-privilege account walks itself to a reverse shell on the gateway box. Obsidian rates the chain CVSS 9.9, and it's hard to argue the number is inflated: a takeover here exposes every provider API key the gateway holds, the secrets used to decrypt stored credentials, and the full content of every prompt and response that has passed through it.

The detail worth sitting with is what Obsidian calls the "man-in-the-gateway" scenario. Because the attacker controls the proxy, they can rewrite model responses before they reach the client — including injecting tool calls that weren't in the original output. If that gateway is sitting in front of an agent like Claude Code, the attacker isn't just reading secrets anymore; they're puppeteering the agent's next action on whatever machine it's running on. That's a fundamentally different risk profile than "API gateway leaks some keys." It means the trust boundary between "model output" and "things the agent will act on" only holds if every hop between the model and the agent is honest, and a compromised LiteLLM instance breaks that assumption silently.

The fixes themselves were not rushed out overnight — BerriAI, LiteLLM's maintainer, had been shipping pieces of the remediation since February, with the full set landing in **v1.83.14-stable** on May 2. Obsidian's disclosure on June 11 and [The Hacker News' coverage](https://thehackernews.com/2026/06/litellm-vulnerability-chain-lets-low.html) on June 15 are really about making sure the install base catches up, not a fresh zero-day. If you're running an older LiteLLM proxy in front of agents that take real actions, this is the kind of patch gap that's worth checking today rather than at the next maintenance window — the value of compromising one of these gateways has gone up considerably now that what's downstream of them can act in the world, not just answer questions.
