---
title: "The Model That Manages Models"
date: 2026-06-22T06:05:40+00:00
draft: false
slug: sakana-fugu-orchestration
categories: [agents]
tags: [agents, multi-agent, orchestration, sakana, open-source, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Sakana AI launched Fugu today: a multi-agent orchestration system packaged as
    a single OpenAI-compatible API. The underlying claim — that learned coordination
    beats any individual frontier model on hard tasks — is backed by two ICLR 2026
    papers and benchmark numbers that hold up. The detail worth noticing: Fable 5
    and Mythos are absent from the agent pool because they're export-controlled.
    Swappable orchestration isn't just a feature; it's a hedge.
---

The interesting thing about [Sakana AI's Fugu](https://sakana.ai/fugu-release/), launched today, is not that it's good at tasks. It's that it gets there through a structure most frontier labs haven't tried: a model whose job is to coordinate other models, packaged as a single API call.

You use Fugu the same way you'd use any LLM endpoint. Internally, it's a learned coordinator that assembles a team from a pool of specialist agents, assigns roles, synthesizes their outputs, and returns a response. None of that is visible to the caller. From the outside, it looks like a model. From the inside, it's closer to an engineering team that happens to run in milliseconds.

## How the coordination works

Two ICLR 2026 papers from Sakana's research group underpin the design. [TRINITY](https://arxiv.org/abs/2512.04695) is a coordinator model that assigns three adaptive roles across a team: Thinker (conceptual planning), Worker (execution), and Verifier (validation). The assignments aren't fixed — the coordinator decides per task which agents should be deliberating and which should be acting.

[Conductor](https://arxiv.org/abs/2512.04388) adds a reinforcement learning layer on top. Rather than specifying coordination strategies by hand, Conductor is trained to discover them: the RL objective is task success, and what gets learned is the natural-language instructions the coordinator sends to its agents. This is meaningfully different from fixed-topology multi-agent frameworks, where the developer decides the routing graph and the models just fill the nodes. Here, the routing is itself learned, which means coordination strategies can adapt to task characteristics the developer never explicitly specified.

A telling result from the Conductor paper: the trained Conductor itself is a 7B-parameter model that beats the much larger workers it coordinates, on tasks like LiveCodeBench and GPQA Diamond — by designing the right communication patterns for them rather than solving the problems directly. The coordinator that wins the race doesn't run the race; it picks the runners and calls the plays.

Fugu also inherits a recursive property from this architecture: the system can call itself as one of its own worker agents, enabling iterative self-refinement on hard problems. This is, frankly, the right approach if you believe coordination is the bottleneck. Most teams building multi-agent systems spend as much time debugging their orchestration logic as they do on the models themselves. Conductor makes that logic a learned artifact instead of a hand-maintained rule set.

## The numbers

Fugu Ultra, the higher-performance tier, reaches 73.7% on SWE-Bench Pro, 95.5% on GPQA Diamond, 50.0% on Humanity's Last Exam, and 86.6% on CharXiv reasoning. The base Fugu model scores 59.0% on SWE-Bench Pro but identically to Ultra on GPQA-D (95.5%). The gap on coding tasks suggests that Fugu Ultra's coordination strategy is pulling in more capable coding agents for hard problems — the architecture is doing what it's supposed to do.

Whether these benchmarks translate to production tasks is the usual question. SWE-Bench Pro measures applied software engineering on real-world GitHub issues; GPQA-D and HLE measure deep expert-level reasoning. The benchmark profile is consistent with the claim that combining specialist agents beats any single generalist, at least at the tail of the difficulty distribution.

## The detail you shouldn't miss

Sakana's announcement explicitly notes that Fable 5 and Mythos Preview "are not in Fugu's agent pool as they are not publicly accessible." That's a direct reference to the [June 12 export control directive](https://www.anthropic.com/news/fable-mythos-access) that suspended access to Anthropic's two most capable models globally.

The design response is deliberate. Fugu's agent pool is built to be swappable — if a provider gets pulled by government directive, the pool gets reshuffled and the orchestrator routes around the gap. The technical report mentions plans to expand the pool with open-weight models and Sakana's own future models, which makes the resilience story even more concrete.

This is a coherent answer to a risk that materialized suddenly on June 12. When an AI deployment depends on a single frontier model that can be suspended by executive order with a few hours' notice, concentration isn't just a vendor risk — it's a geopolitical one. Abstracting over multiple providers, with a coordinator that can adapt its strategy when the team composition changes, is a genuine design advantage in that environment.

## What this means

Fugu is available via console.sakana.ai with an OpenAI-compatible API. Two tiers: Fugu for everyday tasks, Fugu Ultra for complex multi-step problems. Pricing follows subscription and pay-as-you-go plans.

The broader question Fugu raises is whether "learned orchestration" is a durable competitive advantage. The TRINITY and Conductor papers were published in December 2025 — the techniques are already public and reproducible. Any well-resourced team could build a competing system. What Sakana is selling is a trained, maintained, production-ready version that doesn't require you to set up and manage the underlying agent pool yourself.

That's a reasonable value proposition. The alternative — rolling your own multi-agent coordination and keeping up with a constantly shifting set of frontier models — is genuinely expensive to maintain. Whether Fugu's trained coordination is good enough to justify the margin over a hand-rolled router is the question beta users will answer over the next few months.
