---
title: "The Token Relay Economy"
date: 2026-08-17T06:23:47+00:00
draft: false
slug: the-token-relay-economy
categories: [security]
tags: [security, inference, gray-market, openai, anthropic, china]
params:
  author: AI Beat Desk
  summary: >-
    A Vectoral research series maps the gray-market supply chain that routes
    Western AI model access to Chinese buyers at discounts up to 97.8% off
    official pricing. The infrastructure is four layers deep, runs on
    open-source gateways, and poses a model-distillation risk that goes beyond
    simple revenue loss for providers.
---

If you have ever wondered where the cheap AI credits come from — the ones advertised on Telegram at "70% off Claude API," or the ones showing up on gray-market forums with implausible discounts — Vectoral has written the clearest account of the supply chain I have seen. Two posts, [published in late June](https://vectoral.com/blog/token-relay-market) and [updated in early August](https://vectoral.com/blog/who-are-the-token-brokers), document what they call the token relay market: the organized underground infrastructure for routing Western model access to buyers in jurisdictions where U.S. AI providers are unavailable or prohibitively expensive.

The supply chain runs in four stages. At the top sit **card and account merchants**: suppliers of virtual credit cards designed to pass U.S. and European billing verification, plus bulk-registered accounts at major AI providers. Below them are **pool aggregators**, which buy from multiple upstream merchants, aggregate credentials into pools of hundreds of accounts, manage authentication tokens and rate-limit failover, and expose a unified API surface. Then come the **relays** — consumer-facing services, typically in Chinese — that wrap the pool APIs with billing dashboards and support. At the bottom are end users: developers, startups, and AI labs seeking cheap inference or conducting model distillation.

The glue holding the midstream together is two open-source projects: **one-api** and **new-api**, both OpenAI-compatible gateways that "expose a single endpoint that matches the OpenAI API" and handle user management, quotas, and per-request markup as configuration. The fact that the core routing infrastructure is publicly available means the barrier to standing up a relay operation is low. You mostly need upstream credentials and customers.

The economics are striking. Vectoral found packages offering $3,333 in Anthropic API credit for 425 RMB — roughly $58 at current exchange rates. That is approximately a 98% discount from official API pricing. Even allowing for accounting tricks (reselling credits at a loss to acquire usage data, for instance), those numbers imply upstream credit acquisition costs that are far below what legitimate API purchasers pay. The most plausible explanations are a mix of stolen payment credentials, fraudulent account signups, and arbitrage on Claude Max subscriptions, which offer substantially more value per dollar than direct API access.

The revenue loss for providers is real but probably not the most important risk here. What Vectoral flags — and what I think deserves more attention — is the **distillation angle**. A relay service that sits in the request path between a client and a Western frontier model sees every prompt and every response. That is valuable training data. Some relay operators are almost certainly selling that conversation history to labs building models, or using it themselves. The gray market is not just a billing problem; it is a channel through which proprietary model behavior gets extracted at scale by parties who never negotiated a data sharing agreement.

There is also the question of what the user actually gets. A relay at the downstream layer might substitute a cheaper Chinese model for the requested OpenAI or Anthropic model without disclosure. The client sees the right output format; the relay operator pockets the difference. Developers using unofficial relays have no way to verify they are talking to the model they think they are.

The demand side of this market is, structurally, a function of two things: geographic restrictions that prevent legitimate access, and the price differential between subscription and API pricing. Claude Max subscriptions are priced for end users doing casual use, not for developers extracting billions of tokens. The delta between what a subscription costs per token and what the API costs per token is large enough to make resale economically interesting even before you get to stolen credentials. Providers created this arbitrage by not offering equivalent subscription-to-API conversion paths.

With [Stripe acquiring OpenRouter](https://ai-beat.github.io/news/2026/08/stripe-buys-the-router/), the official routing layer is moving toward tighter compliance and geographic enforcement. That will likely push more demand into relay channels rather than less. The underground infrastructure documented by Vectoral is not going away; it is scaling to meet the market that U.S. providers are declining to serve.
