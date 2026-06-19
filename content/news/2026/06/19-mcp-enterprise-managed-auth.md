---
title: "MCP Gets Its Enterprise Authorization Layer"
date: 2026-06-19T08:06:00+02:00
draft: false
slug: mcp-enterprise-managed-auth
categories: [infrastructure]
tags: [mcp, oauth, enterprise, identity, authentication, agents]
params:
  author: AI Beat Desk
  summary: >-
    The Model Context Protocol stabilizes Enterprise-Managed Authorization:
    organizations configure MCP server access once through their identity provider
    and users get zero-touch provisioning via an Identity Assertion JWT flow,
    no per-server consent screens. Okta is the first supported IdP, with Claude,
    Claude Code, and VS Code 1.123 as the first clients. It's the plumbing that
    turns MCP from a developer prototype into something an enterprise can actually
    operate.
---

The Model Context Protocol has had a credibility gap since it launched: it described a clean way for AI assistants to talk to tools, but every connection required a separate per-user OAuth flow. Roll it out to a 500-person engineering org with twelve MCP connectors and you have six thousand individual consent screens to manage — none of them visible to IT, none of them revocable in bulk.

[Enterprise-Managed Authorization](https://blog.modelcontextprotocol.io/posts/enterprise-managed-auth/), published June 18 as a stable MCP extension, closes that gap. The mechanism is called **ID-JAG** — Identity Assertion JWT Authorization Grant. Instead of each user completing a separate OAuth round-trip per MCP server, the client obtains an identity assertion JWT from the organization's identity provider during normal single sign-on, then presents that JWT to the MCP server's authorization endpoint in exchange for an access token. The user sees no consent screen. The MCP server knows who the user is and what they're authorized to access, derived from the groups and roles they already carry in the IdP.

Okta is the first supported identity provider, using its [Cross App Access](https://developer.okta.com/docs/concepts/mcp-server/) (XAA) protocol as the integration point. The admin workflow is: configure the MCP server as a connected app in Okta, assign it to the relevant user groups, and you're done. Users accessing Claude, Claude Code, or VS Code 1.123 — the three clients shipping support at launch — get the configured servers available immediately on their next login.

The list of MCP services supporting EMA at launch is notable: Asana, Atlassian, Canva, Figma, Granola, Linear, and Supabase are in, with Slack coming. These aren't toy integrations — this covers the project management, design, and collaboration tools that form a large share of what knowledge workers actually need connected to their AI assistant. The fact that Anthropic and Microsoft are implementing on the client side simultaneously, rather than one of them dragging, matters too. Protocol specs without reference implementations are just documents.

The technical design makes a reasonable choice about trust boundaries. The ID-JAG isn't just a renamed access token — it's a short-lived assertion that the IdP signs about the user's identity and group membership. The MCP server can validate this cryptographically and derive access decisions from it, without needing to call back to the IdP for every request. This is federated identity done properly: the MCP server participates in the organization's trust fabric rather than maintaining its own user database.

From a threat model perspective, the interesting change is that authorization policy now lives in the IdP rather than scattered across individual MCP server configurations. An employee leaving the company gets their MCP access revoked when their IdP account is suspended — automatically, consistently, with no manual cleanup across twelve separate tool integrations. That's the operational improvement IT actually cares about.

What's still missing is a multi-IdP story. Okta first is a reasonable starting point given its enterprise market share, but organizations running Azure AD, Google Workspace, or Keycloak need their own implementations. The extension spec is open; the question is who builds those connectors and on what timeline.

The bigger picture: MCP spent its first year as a protocol that developers found interesting and enterprises found impractical. Enterprise-Managed Authorization is a necessary step toward the latter being able to say yes. It won't be the last — logging, auditing, and data-residency controls are all still open questions — but it's the one that clears the first gate.
