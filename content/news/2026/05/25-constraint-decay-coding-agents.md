---
title: "When Constraints Stack, Agents Stumble"
date: 2026-05-25T06:15:00Z
draft: false
slug: constraint-decay-coding-agents
categories: [agents]
tags: [agents, code-generation, benchmarks, production, web-frameworks]
params:
  author: AI Beat Desk
  summary: >-
    A new paper studies what happens to LLM coding agents as structural
    requirements accumulate in backend tasks — architecture constraints,
    ORM rules, database schemas. The answer is a ~30 percentage-point drop
    in test pass rates from baseline to fully specified tasks, with database
    constraints alone responsible for 19pp of that. Flask agents do fine;
    Django and FastAPI agents do not.
---

There's a gap that's been obvious to anyone who has tried to use a coding agent for production backend work, and a new paper called [Constraint Decay](https://arxiv.org/abs/2605.06445) now has the numbers to describe it precisely.

The setup: evaluate LLM agents on 80 greenfield and 20 feature-implementation tasks across eight web frameworks (Flask, FastAPI, Django, aiohttp on Python; Express, Fastify, Hono, Koa on Node.js). Tasks are organized into four constraint levels — L0 is purely functional with no structural requirements, and L3 is fully specified with explicit architectural patterns, ORM constraints, and database schema rules. The metric is Assert% (A%): the fraction of behavioral test assertions passed across all runs.

The finding is sharp. Capable model configurations lose roughly **30 percentage points** on average moving from L0 to L3. Weaker configurations sometimes approach zero. The leading contributor is database constraints, which account for an average 19.3pp of that decline — wrong query composition, ORM runtime violations, subtle mismatch between the model's learned pattern and the actual schema. Architecture constraints add another 9.1pp. ORM specification alone has minimal impact on its own; it's the interaction with database constraints that bites.

The framework sensitivity is equally striking. Agents succeed at around 50% A% on minimal, explicit frameworks — Flask, Express, Koa. On convention-heavy ones — FastAPI and Django — average performance drops to around 24%. This makes a kind of sense: Flask gives you a blank page, and the model fills it in with patterns it knows well. FastAPI's dependency injection, Pydantic model binding, and async lifespan conventions are implicit, learned from training data but fragile under pressure. Django's ORM is deeply convention-driven, and getting it right requires not just generating code but maintaining a coherent model of what the ORM expects at runtime.

The models studied range from Devstral-Small (24B) and Qwen3-Coder-Next (80B) to frontier-class models like GPT-5.2 and MiniMax-M2.5. All degrade as constraints accumulate; the gap between strong and weak configurations narrows at L3 rather than widening, which suggests the failure mode is structural rather than a matter of raw capability.

What does this mean practically? The paper draws the line between rapid prototyping and production-grade software. In the former, an agent's ability to produce plausible-looking, runnable code is enough. In the latter, production code must satisfy formal architectural constraints — framework conventions, ORM contract expectations, schema-level correctness — simultaneously. These aren't optional. A database query that's syntactically valid but violates a constraint the ORM enforces at runtime will silently fail or crash under load. The agent doesn't know this; it generates code that looks correct and often is correct in isolation.

There's a subtler point the paper surfaces too. The best agents in the study were evaluated on framework versions from roughly six months before submission. Comments in the HN thread note this, and it's fair. But the finding doesn't seem like it should be model-generation-dependent: the underlying issue is that as you stack explicit requirements, each one narrows the valid solution space and increases the chance that the agent satisfies some constraints at the expense of others. That's a combinatorial problem that gets harder with scale, not one that frontier models have obviously solved.

The evaluation pipeline, task suite, and analysis scripts are available in the paper's repository. For anyone building production backend code with agents — or evaluating whether to — the constraint taxonomy and the per-framework breakdown are worth examining closely.
