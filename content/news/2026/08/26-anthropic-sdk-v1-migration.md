---
title: "Anthropic's SDK Hits 1.0: The Knobs That Moved Server-Side"
date: 2026-08-26T06:15:00+00:00
draft: false
slug: anthropic-sdk-v1-migration
categories: [tooling]
tags: [anthropic, sdk, python, api, migration]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic's Python SDK crossed its first major version boundary on August 20,
    dropping fifteen breaking changes including the removal of temperature, top_p,
    and top_k from the Messages method signatures. The removals aren't just
    cleanup — newer Claude models already reject non-default sampling values with
    HTTP 400, and context compaction has moved server-side. The SDK is catching up
    to what the API already enforces.
---

The [Anthropic Python SDK](https://github.com/anthropics/anthropic-sdk-python/releases/tag/v1.0.0) crossed its first major version boundary on August 20, moving from v0.125.0 to v1.0.0 with fifteen documented breaking changes. Most of them are straightforward cleanup: httpx replaced by httpx2, Python 3.10 becomes the minimum, byte-value headers no longer accepted, some type aliases renamed. These are the boring-but-necessary housekeeping you'd expect at any API 1.0 milestone.

The removals that aren't boring are the ones that reflect what newer Claude models already enforce.

Starting with v1.0.0, `temperature`, `top_p`, and `top_k` are gone from the Messages method signatures. This isn't purely an SDK design decision — seven current Claude models return HTTP 400 when you try to set non-default sampling values. The SDK is catching up to what the API already does. Anthropic's newest models don't expose raw sampling knobs to clients. Sampling happens internally, and the model's judgment about temperature, nucleus-sampling threshold, and top-k cutoff isn't a client concern anymore.

Context compaction tells the same story. The client-side `compaction_control` parameter on tool runners is removed. Compaction moves to a server-side beta feature (`compact-2026-01-12`). What used to be something a client could configure directly is now a server operation that clients can opt into but not tune.

Put together, the pattern is clear: Anthropic is pulling inference internals server-side. Clients used to reach directly into sampling and compaction behavior. The newer model generation either ignores those parameters or rejects them outright. The SDK is making explicit what the API already enforces.

The other significant removal is the legacy Text Completions API. `client.completions.create()` is gone, along with the `HUMAN_PROMPT` and `AI_PROMPT` constants that came with it. These date back to the original Anthropic API design — the `\n\nHuman:\n\nAssistant:` prompt format that predated Messages. It's been deprecated for a long time; v1.0.0 finally deletes it.

On the runtime side: `AnthropicBedrock` now raises an error if no AWS region is configured, rather than silently defaulting to us-east-1. Async raw responses require `await response.parse()` instead of working synchronously. The `output_format` schema parameter on parse helpers moved to `output_config`. None of these are surprising, but all of them will break existing code if you haven't read the [migration guide](https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md).

v0.125.0 remains available as a hold-back pin for teams that can't migrate immediately. For everyone else, the migration guide covers all fifteen changes with before-and-after snippets.

The 1.0.0 designation means Anthropic considers the API surface stable — future breaking changes should increment the major version. Whether that holds depends on how quickly their models continue to evolve. Given that at least seven current models already silently disagree with what clients can configure, "stable" here means the client-visible surface, not the underlying behavior.

The deeper takeaway is architectural. For the first eighteen months or so of the Messages API, the client controlled a lot: sampling parameters, whether to compact context, which region an inference request hit. The server was fairly transparent. The current generation of Claude models is less transparent in exactly these areas — they make decisions that used to be client-configurable, and they enforce those decisions with hard errors rather than silently ignoring parameters they disagree with. The SDK has finally caught up to that reality.
