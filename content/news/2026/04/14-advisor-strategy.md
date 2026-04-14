---
title: "The Advisor in the Room"
date: 2026-04-14T06:10:00Z
draft: false
slug: advisor-strategy-anthropic-api
categories: [agents]
tags: [anthropic, api, agents, cost, architecture]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic's new advisor tool formalizes a pattern that practitioners have
    been assembling by hand: a fast executor model (Sonnet or Haiku) that can
    consult Opus for strategic guidance mid-generation, entirely server-side
    within a single API call. The benchmarks show real gains and the
    implementation is notably clean — but the more interesting shift is
    architectural: it treats Opus-level intelligence as a resource to be
    invoked selectively rather than paid for on every token.
---

On April 9, Anthropic published [the advisor strategy](https://claude.com/blog/the-advisor-strategy), a blog post and API feature that does something straightforward but with meaningful implications for how agent systems get built. The short version: a fast, inexpensive executor model (Sonnet 4.6 or Haiku 4.5) can call Opus 4.6 mid-generation for strategic guidance, entirely within a single `/v1/messages` request, with no extra round-trips on the client side.

The "cheap executor, expensive advisor" pattern isn't new. People have been assembling versions of it manually — running two separate API calls, passing summaries between models, building their own orchestration logic. What's new is that Anthropic has formalized it as a first-class API primitive, and the server-side implementation is notably cleaner than what you'd cobble together yourself.

## How it works

You add an `advisor_20260301` entry to your `tools` array alongside whatever other tools your agent uses. When the executor reaches a decision it wants help with, it invokes the advisor like any other tool — but the input is always empty. The server constructs the advisor's view automatically from the executor's full conversation transcript: system prompt, all tool definitions, all prior turns, all tool results. The advisor runs a separate inference pass, returns a plan (typically 400–700 text tokens, 1,400–1,800 tokens total including extended thinking), and the executor continues.

The advisor never calls tools. It never produces user-facing output. It reads everything, writes a plan, and hands control back. That constraint is worth sitting with: the advisor is a planning resource, not a parallel agent. Its thinking blocks are dropped before the result returns to the executor; only the advice text comes through.

From an implementation standpoint, the whole thing runs inside one request. No extra round-trips, no client-side orchestration, no session state to manage between models. The [API docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) are unusually detailed and cover a bunch of edge cases worth knowing — including a subtle interaction where `clear_thinking` with anything other than `keep: "all"` breaks the advisor's prompt cache by shifting the quoted transcript on each turn.

To try it in Python with the current Anthropic SDK:

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    betas=["advisor-tool-2026-03-01"],
    tools=[
        {
            "type": "advisor_20260301",
            "name": "advisor",
            "model": "claude-opus-4-6",
        }
    ],
    messages=[{"role": "user", "content": "..."}],
)
```

## The numbers

Benchmark results are encouraging in the places where they should be. Sonnet 4.6 with an Opus advisor went from 72.1% to 74.8% on SWE-bench Multilingual — a 2.7 percentage point gain — while cost per task dropped 11.9% (because the advisor generates a short plan while the executor handles the bulk of output at Sonnet rates). That's a modest gain for a complex coding task; not transformative, but real.

The more dramatic result is on BrowseComp, a multi-step web research task: Haiku 4.5 alone scored 19.7%, while Haiku with an Opus advisor scored 41.2% — more than double — at 85% lower cost than just switching to Sonnet. That gap suggests the benchmark rewards good planning heavily, and that Haiku's main weakness on that task is strategic rather than generative. When Opus provides the strategy, Haiku can execute it.

Results are task-dependent, and the docs say so clearly. The advisor is a weak fit for single-turn Q&A (nothing to plan) or workloads where every turn genuinely requires the advisor model's full capability. The fit is strongest for long-horizon agentic tasks where most turns are mechanical but the overall approach needs to be good from the start.

## Why this matters architecturally

The pattern this encodes — use cheap compute for most of the work, expensive intelligence only for key decisions — is the right instinct for production agent systems. But most teams implement it badly, with ad hoc two-model calls and leaky state management. The advisor tool gives you that pattern with clean semantics, server-managed context passing, and billing you can actually reason about (advisor tokens appear in `usage.iterations[]` separately from executor tokens, billed at the advisor model's rate).

There's also a useful cost-control primitive in `max_uses`. You can cap how many advisor calls a single request makes — if you set `max_uses: 2`, the executor gets two advisor consultations and then handles the rest itself, getting a `max_uses_exceeded` error gracefully and continuing. Combined with the suggestion in the docs to prompt the executor to call the advisor once near the start and once before declaring done, you have a natural way to bound Opus exposure without giving up the gains.

The streaming behavior is worth noting for anyone building UX on top: the executor stream pauses while the advisor runs (server-side inference, nothing sent to the client except SSE keepalive pings every ~30 seconds). Short advisors may show no pings at all. For fast-paced interactive agents this could feel like a hang; for background agentic tasks it's irrelevant.

## The advisor prompt caching wrinkle

There are two independent caching layers: executor-side (standard `cache_control` on content blocks) and advisor-side (a `caching` parameter on the tool definition itself). The advisor-side cache is an on/off switch, not a breakpoint marker — you enable it for a conversation and the server handles cache boundaries. It breaks even at roughly three advisor calls per conversation and improves from there.

The gotcha: if you use extended thinking with its default `clear_thinking` behavior (`keep: {type: "thinking_turns", value: 1}`), the advisor's quoted transcript shifts on each turn, causing cache misses. Set `keep: "all"` to preserve advisor cache stability when you want caching to work properly across a long agent loop. It's an easy thing to miss when you're first wiring this up.

## In context

The advisor tool is in beta, requires a specific header (`anthropic-beta: advisor-tool-2026-03-01`), and needs account team access. That's a narrower rollout than a general release, but the API docs are detailed enough that it's clearly meant to be used by real production systems, not just as a preview. The pattern — formalize something practitioners are already building, make it server-managed, expose clean usage accounting — is consistent with how Anthropic has been evolving its API surface.

Whether you need it depends on what you're building. For short-context tasks or single-turn use, it doesn't add much. For long-horizon coding agents or multi-step research pipelines where the plan matters as much as the execution, it's worth evaluating seriously.
