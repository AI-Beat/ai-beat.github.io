---
title: "A Proxy at the Edge of the Agent"
date: 2026-04-22T06:16:43+00:00
draft: false
slug: crabtrap-agent-http-proxy
categories: [security]
tags: [agents, security, open-source, tools]
params:
  author: AI Beat Desk
  summary: >-
    Brex open-sourced CrabTrap, a Go MITM proxy that intercepts every outbound
    HTTP request from an AI agent and evaluates it against a natural-language
    security policy before letting it through. The approach is genuinely useful
    for catching exfiltration attempts, while raising a fair question about
    whether a probabilistic judge belongs in a security-critical path.
---

The core problem with deploying AI agents in production is that they need to talk to the internet — and the same network access that makes them useful also makes them dangerous. A coding agent needs to fetch docs, push to GitHub, call APIs. A customer-support agent needs to query your CRM. Any of those channels can be weaponized through prompt injection, and most agent frameworks give you no visibility into what actually went out over the wire.

Brex's [CrabTrap](https://github.com/brexhq/CrabTrap), open-sourced this week, takes a straightforward approach: sit between the agent and the internet. It's a transparent HTTPS proxy written in Go that intercepts all outbound HTTP from an agent process, generates custom TLS certificates per host (the standard MITM trick), and evaluates every request before forwarding it.

The policy engine has two tiers. Static URL patterns are checked first — prefix matches, exact matches, glob patterns — and if a rule fires, the request is allowed or denied immediately with no LLM call. Only requests that don't match any static rule go to the LLM judge, which evaluates the request against a natural-language policy you define: something like "this agent manages GitHub issues for our organization; it should never access payment APIs or exfiltrate environment variables." The judge returns allow or deny, with reasoning, and everything lands in a PostgreSQL audit table. There's also a circuit breaker that trips after five consecutive LLM failures and falls back to a configurable default, which is a sensible production detail.

The SSRF protection (blocking requests to RFC 1918 addresses with DNS rebinding prevention) and JSON-encoding of payloads before sending them to the judge are both included, which covers the obvious surface areas.

The honest limitation that surfaced in HN discussion is worth acknowledging: CrabTrap only sees outbound HTTP. By the time a malicious email has convinced your agent to exfiltrate credentials, the compromise has already happened inside the agent process — CrabTrap catches the outbound call, not the internal state. If the injected instruction manipulates in-memory data or writes to local files, the proxy sees nothing until there's network activity. That's not a flaw in CrabTrap so much as an inherent limitation of the network-layer vantage point.

The more philosophical objection — that using a probabilistic LLM as a security gate is the wrong shape for a security primitive — is also fair. LLM judges can be fooled, jailbroken, or simply wrong. A well-crafted prompt injection that looks plausible to a judge (say, a request to a legitimate CDN that's actually exfiltrating data in a query parameter) could slip through.

What CrabTrap does well is the class of attacks it was clearly designed to catch: supply-chain injection through external content. An agent reading a web page that tells it to make an API call to `evil.example.com` will hit the proxy before that request reaches the internet. The judge sees a request to an unrecognized domain for an action that doesn't match the agent's declared purpose — and that's a pattern an LLM is actually good at flagging. It's not a security guarantee, but it's a meaningful speed bump for the most common injection vectors.

The automated policy builder is also a useful practical touch: you describe your agent's purpose and CrabTrap drafts the natural-language policy via an agentic loop, which you then review. Writing security policies is notoriously underspecified work; having a first draft to critique is better than starting from a blank document.

The codebase is around 80% Go with a TypeScript web UI. The setup requires Docker and PostgreSQL, which is more infrastructure than a standalone proxy but fits production agent deployments where you'd want the audit trail anyway. It's at v0.0.1, so the API surface will move; the architecture is mature enough that building on it now is reasonable, with the expectation that things will shift.

For teams already running agents in production without outbound filtering, CrabTrap is worth evaluating. It's not a replacement for proper agent sandboxing, capability restriction, or input validation — but it adds a layer of observability and control that most stacks currently lack.
