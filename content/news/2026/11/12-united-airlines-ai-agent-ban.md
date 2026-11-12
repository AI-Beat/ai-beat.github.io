---
title: "Airlines and the Access Problem"
date: 2026-11-12T06:17:30+00:00
draft: false
slug: united-airlines-ai-agent-ban
categories: [agents]
tags: [agents, policy, deployment]
params:
  author: AI Beat Desk
  summary: >-
    United Airlines updated its booking policy on November 11 to prohibit
    third-party AI agents from making reservations on behalf of travelers,
    citing security, unintended purchases, and federal luggage rules.
    Southwest takes the opposite stance. The divergence reveals a split in
    how airlines think about agent access — and raises questions about whether
    commercial friction will shape what AI agents can actually do in the real world.
---

On Wednesday United Airlines [updated its booking and ticketing policy](https://www.cbsnews.com/news/united-airlines-bans-ai-agents-booking-flights/) to prohibit what it calls "unauthorized, third-party personal AI agents or assistants" from making flight reservations on behalf of travelers. The policy applies to its own digital platforms and to third-party travel aggregators.

Three reasons are given. First, security: using an AI agent means giving it account credentials and payment information, and United warns that credential sharing exposes sensitive data. Second, unintended actions: agents could make purchases, change itineraries, or spend loyalty miles in ways the traveler didn't intend. Third — and this one is specific to airlines — federal regulations require travelers to personally verify that their bags contain no prohibited items during the check-in process. An AI agent checking you in can't do that.

The first two reasons could apply to almost any service that handles money or identity. The third is genuinely specific to aviation, and it's interesting because it isn't a decision airlines made: it's a regulatory constraint that predates agentic AI by decades and now creates a structural barrier to full agent delegation. You can delegate the booking, but check-in remains a human step, which means agent-managed travel always bottlenecks on a person.

Southwest's approach is notably different. The airline has kept its inventory visible to ChatGPT users, apparently treating AI-assisted browsing as a customer acquisition channel rather than a threat. Aviation analyst Henry Harteveldt points out the other side of United's reasoning: a surge in AI-driven flight searches — agents probing availability for many potential itineraries — imposes real infrastructure costs without necessarily converting to bookings. Blocking agents also keeps United in control of its distribution channel, which matters commercially when many airlines are trying to reduce dependence on third-party booking platforms.

What this reveals is less a principled stance on AI and more a business calculation dressed up in safety language. The security arguments are real but not unique to AI — credential-sharing risks apply to any third-party app that gets account access. The distribution-control argument is probably the stronger driver. Airlines have spent years trying to route bookings through their own channels, and AI agents are a new pressure point in that ongoing fight.

The timing is interesting given that Google launched the [Gemini Decisions API](https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/) the same day — an endpoint that lets developers run up to 1,000 structured evaluation questions against a multimodal input in a single forward pass. The API is explicitly aimed at making agents better at structured decision-making: does this booking match the user's preferences, is this fare acceptable, should I proceed. The infrastructure for agents to be more capable is arriving at the same time as the policies restricting where they can act.

That gap — between technical capability and permitted access — will probably define a lot of the next few years for deployed AI agents. The interesting battles aren't going to be about whether an agent can book a flight; they're going to be about who controls the credentials, who bears liability when something goes wrong, and which intermediaries have the contractual leverage to block agent access when it threatens their margins.
