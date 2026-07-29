---
title: "MCP's Stateless Pivot"
date: 2026-07-29T06:11:36+00:00
draft: false
slug: mcp-stateless-2026-spec
categories: [tooling]
tags: [mcp, tooling, infrastructure, protocol, agents]
params:
  author: AI Beat Desk
  summary: >-
    The 2026-07-28 MCP specification drops protocol-level sessions entirely,
    making the HTTP transport stateless: every request now carries its own
    protocol version and client capabilities, enabling standard load balancers
    without sticky sessions. The revision also introduces Multi Round-Trip
    Requests to replace server-initiated streams, adds cacheable list results,
    and formally deprecates Roots, Sampling, and Logging.
---

The Model Context Protocol's [2026-07-28 specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/) shipped Monday, and its headline change — removing sessions from the HTTP transport — is the most significant revision to MCP's wire format since the protocol launched.

The original Streamable HTTP transport required an `initialize`/`notifications/initialized` handshake before any real work could happen, and the server issued a session ID (`Mcp-Session-Id`) that clients had to carry on every subsequent request. The consequence was that a client that connected to server instance A during initialization couldn't have its follow-up requests land on server instance B — you needed sticky sessions, a shared session store, or both. Running MCP in production meant working around the protocol's own architecture: extra infrastructure for session affinity, shared-memory backends to replicate state, careful gateway configuration. For a protocol that's supposed to be the substrate for tool integrations at scale, this was a persistent friction point.

The 2026-07-28 spec removes all of it. Every request now carries its protocol version and client capabilities in `_meta` fields (`io.modelcontextprotocol/protocolVersion`, `io.modelcontextprotocol/clientCapabilities`). No handshake. No session ID. Any request can land on any server instance, which means plain round-robin load balancing works without modification. A new `server/discover` RPC lets clients probe compatibility before sending substantive requests — useful for backward-compatibility checks with servers that haven't upgraded yet.

The second major change is Multi Round-Trip Requests (MRTR), which replaces the pattern where servers could initiate back-channel requests to clients — for things like sampling (asking the client to run an LLM), elicitation (requesting user input), or querying filesystem roots. Under the old design, this required an open stream maintained by the server, which made servers stateful even when they didn't logically need to be. Under MRTR, a server that needs additional input returns `resultType: "input_required"` with the information it needs, and the client retries the original request with an `inputResponses` field. It's slightly more verbose but fully stateless: the server doesn't need to remember anything between the initial response and the retry. The removal of SSE stream resumability follows the same logic — broken streams force a retry rather than a redelivery, which is cleaner under concurrent load.

Three features are now formally deprecated: Roots (the mechanism for clients to advertise filesystem paths), Sampling (server-initiated LLM inference via the client), and Logging (server-to-client log forwarding). The deprecation window is at least twelve months, so nothing breaks immediately. But the direction is clear. Roots and Sampling were designed for a world where MCP clients and servers were assumed to be colocated with an interactive user; they don't compose well in multi-tenant or server-to-server configurations. Sampling in particular was always a conceptually odd feature — a server asking a client to run a model and return results, essentially using the MCP session as a proxy for the LLM API. The honest replacement is for servers to talk to LLM APIs directly.

Some of the minor changes are worth noting. List operations (`tools/list`, `resources/list`, `prompts/list`) now return `ttlMs` and `cacheScope` fields, which let clients and gateways cache tool listings instead of polling on every request. For deployments with many clients and a stable tool set, this cuts unnecessary chatter. The new `Mcp-Method` and `Mcp-Name` HTTP headers let WAFs and gateways inspect and route requests without parsing JSON bodies — a straightforward improvement for security policy enforcement.

Tier 1 SDKs (TypeScript, Python, Go, C#) support this version from day one, which is notable — in previous revisions there was often a gap between spec and SDK coverage.

None of this is conceptually novel. Stateless HTTP, explicit retry patterns, cache headers — these are infrastructure primitives that web services have used for decades. The interesting thing is that MCP needed a full year and a half of production deployment to learn which parts of its original design didn't survive contact with real infrastructure requirements. The 2026-07-28 spec is the answer. If the 2025 spec was MCP getting things to work, this one is MCP being built to run at scale.
