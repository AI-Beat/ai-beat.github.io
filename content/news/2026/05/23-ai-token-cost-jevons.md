---
title: "Cheaper Per Token, More Expensive Overall"
date: 2026-05-23T06:05:29+00:00
draft: false
slug: ai-token-cost-jevons
categories: [industry]
tags: [industry, agents, inference, llm, cost]
params:
  author: AI Beat Desk
  summary: >-
    Token prices are falling fast, but enterprise AI bills are rising. Uber
    burned through its entire 2026 AI coding budget in four months driven by
    Claude Code adoption. Goldman Sachs projects a 24× increase in token
    consumption by 2030. The Jevons paradox shows up again: efficiency gains
    don't reduce consumption — they expand it.
---

There's a recurring pattern in energy and resource economics: make something more efficient to use, and total consumption goes up rather than down. [William Stanley Jevons](https://en.wikipedia.org/wiki/Jevons_paradox) observed this with coal in the 1860s. Steam engines got more efficient, coal became more economical per unit of work, and total coal consumption grew. The same dynamic is playing out right now with AI inference tokens.

[A Fortune report published Thursday](https://fortune.com/2026/05/22/microsoft-ai-cost-problem-tokens-agents/) aggregates evidence that for a growing number of organizations, the total cost of running AI systems has crossed or is approaching the cost of the human employees those systems were meant to augment. Nvidia researcher Bryan Catanzaro put it directly: "For my team, the cost of compute is far beyond the costs of the employees." That's a striking statement from someone at a company that sells the compute.

The Uber case is the most quantified version of the story available. Uber's CTO disclosed earlier this year that the company exhausted its entire 2026 AI coding tools budget in four months. The primary driver was Claude Code, which has achieved 95% monthly active adoption among Uber's engineering organization — 70% of committed code now originates from AI assistance. Monthly API costs per engineer were running between $500 and $2,000. The finance team, which had presumably modeled AI costs based on earlier, more modest usage patterns, got a surprise.

The per-engineer range matters. $500/month per engineer across a large engineering organization is a line item that competes with infrastructure budgets, not software licenses. $2,000/month starts to approach or exceed a meaningful fraction of fully-loaded employment cost for engineers in some markets. And that's a current number, with models that are used for code completion and review, not for fully autonomous agentic workflows running around the clock.

## Why Falling Prices Don't Help

The standard response to enterprise sticker shock about AI costs is "prices will come down." This is true. Gartner forecasts that inference costs will drop roughly 90% by 2030. The problem is that token consumption won't stay flat while that happens. Goldman Sachs projects a 24-fold increase in token consumption by 2030 as agentic AI workflows replace the chat interfaces that currently dominate AI usage, reaching 120 quadrillion tokens per month across the industry. Ninety percent cost reduction multiplied by 24× consumption is still more than double the total spend.

Agentic systems are the key variable. When a person has a conversation with an AI assistant, the exchange involves bounded token counts — a question, some back-and-forth, a conclusion. When an agent runs autonomously on a task — scanning a repository, drafting a report, monitoring a pipeline — it may execute hundreds of sub-tasks, each consuming a full context window. The token consumption per unit of useful work is orders of magnitude higher than conversational usage. This was always the arithmetic; it's only now becoming visible at scale because agentic adoption is reaching production workloads.

The budgeting problem is arguably as interesting as the cost problem itself. Per-seat software pricing is predictable. You know how many engineers you have; you multiply by the license cost. Consumption-based billing for AI is not predictable in the same way because usage correlates with how aggressively engineers adopt the tools, which correlates with how useful the tools become, which correlates with model quality, which improves over time. An organization that budgets for AI based on current usage patterns and then deploys a better model is likely to underforecast.

## What Changes

None of this is an argument against using AI tools. The productivity gains that are driving 70% of Uber's committed code to come from AI are real and presumably more valuable than the API spend. But the economic model that assumed AI would primarily displace human tasks rather than generating new consumption patterns — infinite todo lists, always-on monitoring, autonomous refactoring — is running into the actual deployment numbers.

The organizations that navigate this well will probably be the ones that treat AI token consumption as a first-class infrastructure metric with the same instrumentation and budget discipline they apply to compute and storage. The ones that don't will keep being surprised.
