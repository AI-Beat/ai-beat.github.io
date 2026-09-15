---
title: "The AI Store Is Open and Losing Money"
date: 2026-09-15T06:09:50+00:00
draft: false
slug: pion-andon-labs-ai-businesses
categories: [agents]
tags: [agents, evaluation, economics, real-world]
params:
  author: AI Beat Desk
  summary: >-
    Andon Labs launched Pion on September 14, a platform that lets AI agents
    run real businesses with access to email, banking, phones, and browsers.
    The company's own experiments — a vending machine at Anthropic's office,
    a retail store in SF burning through $40k in five months, a café in
    Stockholm — offer a more honest evaluation of current AI capability than
    any benchmark.
---

The hardest evaluation for an AI system is one where you can't game the metric. Revenue is like that. The market doesn't grade on a curve and there's no partial credit for coherent reasoning if your store is empty.

[Andon Labs launched Pion](https://andonlabs.com/pion) on September 14 as a research preview: a cloud platform where persistent AI agents operate real businesses. The pitch is empirical rather than speculative — rather than asking whether frontier models *could* run a company, the team has been doing it since 2025 and now wants to find out what happens when more people try with different models and domains.

Their own track record so far is instructive. A vending machine installed at Anthropic's San Francisco office in early 2025 initially bled money as models struggled with basic inventory logic. By late 2025, as underlying model capability improved, the agent reached profitability. That's a meaningful data point: not an impressive demo, but a small machine that makes money.

In April 2026 the experiments got larger. An agent named Luna took over a retail store (Andon Market) in San Francisco. A separate agent runs Andon Café in Stockholm. Neither is profitable today, five months in, and the SF experiment in particular has attracted some [pointed coverage](https://slashdot.org/story/26/09/13/0523208/a-visit-to-san-franciscos-ai-run-store-no-customers-nothing-useful-and-losing-money-fast). Luna stocked the shelves with board games, paperback novels, and novelty gifts — the kind of inventory you'd find at a white elephant exchange, not a neighborhood store. Starting budget: $100,000. Amount burned: around $40,000 over five months, with revenue lagging AI token costs. A journalist visiting on a sunny weekday found essentially no customers and a Rube Goldberg purchasing experience that required picking up a telephone handset mounted on a wooden hand sculpture to complete a soda purchase.

It's easy to read that as an indictment. But the framing Andon Labs offers is more interesting than the failure: simulations don't capture how models behave in the real world. This is a point that's hard to argue with. When you put an agent on SWE-bench, the environment is synthetic, the tasks are drawn from a curated distribution, and there's no feedback loop between the agent's decisions and the problem space. When you put an agent in charge of a retail storefront, the full complexity of the world shows up — supply chains, foot traffic, pricing relative to what's actually in the neighborhood, the gap between what an agent thinks people want and what people actually buy.

Luna's inventory failures are, in their way, information. The agent apparently doesn't yet have good intuitions about what makes a neighborhood convenience store viable versus what sounds plausible as "inventory" to a language model with broad knowledge but limited local economic grounding. That's a specific and addressable failure mode. Knowing it exists is more valuable than a high SWE-bench score.

The Pion platform itself gives agents access to email, phone, banking, and a browser — the same tools a remote human employee would use. The monitoring layer is there because the team is serious about not accidentally creating something that causes real harm at scale. Pion agents still need human approval for some decisions, which is honest: even a profitable AI store operator is probably not a fully autonomous one yet.

The vending machine is the most interesting data point in the whole story. An agent went from losing money in early 2025 to making money by late 2025, with no change in the task or environment, just improvements in the underlying models. That's the kind of capability curve Andon Labs is trying to measure: not a single snapshot of current performance, but how it tracks over time as frontier capabilities shift. Opening it up to external researchers with the Pion platform means that curve can be traced across many more domains and model choices.

The SF store is losing money. But it's losing money in a way that's legible, attributable, and improvable. That might be more than we can say for a lot of things we're currently using AI to optimize.
