---
title: "How a Pool Size Mismatch Leaked 155 LLM Responses Across Tenants"
date: 2026-10-27T06:15:00Z
draft: false
slug: kv-cache-tenant-leak
categories: [infrastructure]
tags: [infrastructure, security, inference, cloudflare, sglang]
params:
  author: AI Beat Desk
  summary: >-
    Cloudflare disclosed a cross-tenant KV cache exposure in its Workers AI
    GLM-5.3 deployment: 155 responses contained data from other customers' requests.
    The root cause was two overlapping bugs—an undersized KV pool that was flagged
    before launch and dismissed, plus an SGLang flaw that had been reported upstream
    months earlier. The incident is a useful case study in how LLM inference
    infrastructure fails differently than traditional web backends.
---

On October 26, Cloudflare [published a detailed incident report](https://blog.cloudflare.com/kv-cache-exposure-remediation/) disclosing a cross-tenant data leak in its Workers AI service. Between August 27 and September 9, 155 responses from the GLM-5.3 model contained data from other customers' requests. Cloudflare was alerted by a researcher from Knostic on September 8, had the model offline within 40 minutes of declaring an incident, patched the system, and restored service the same day. The disclosure came six weeks later, after the fixes were confirmed stable.

The technical cause is worth reading carefully, because it illustrates a failure mode that's specific to modern disaggregated LLM inference and wouldn't arise in a conventional web backend.

## Two bugs, one leak

GLM-5.3 has a 1,048,576-token context window. But when Cloudflare deployed it, the KV cache pool on the prefill GPUs was sized to hold only 681,280 tokens—roughly two-thirds of the maximum context. The team knew about this before launch. They tested with long prompts, got HTTP 200 responses, and concluded the pool size wasn't a problem. The tests checked status codes and latency, not whether the returned text was correct.

The second issue was in [SGLang](https://github.com/sgl-project/sglang), which Cloudflare uses for disaggregated prefill/decode operation. When a prompt exceeded the pool size, SGLang correctly marked the request as rejected—but then swapped it for a one-token placeholder and let it proceed through the admission queue. The queue checked whether a request was finished, not whether it had been rejected. So the rejected request moved forward, prefill sent one KV page, but decode had already reserved eight pages and attended over all of them—including stale pages from previous requests belonging to other tenants.

KV pages are not zeroed between requests. This is an intentional performance choice: zeroing GPU memory is expensive, and ownership tracking is supposed to make it unnecessary. When ownership tracking breaks, the stale pages contain exactly what they look like—fragments of someone else's conversation.

The SGLang bug (issue [#30233](https://github.com/sgl-project/sglang/issues/30233)) had been filed upstream on July 6, more than seven weeks before GLM-5.3 launched. It wasn't a zero-day; it was a known edge case that hadn't been fixed yet. Cloudflare's prefill pool misconfiguration created the conditions to trigger it in production.

## What this means for inference operators

A few things stand out from Cloudflare's post-incident changes that are worth noting for anyone running multi-tenant LLM inference:

**Pool size validation needs to be explicit.** SGLang computes KV pool size from available GPU memory at startup rather than from a configured parameter, so there's no obvious place to enforce that pool capacity ≥ context length. Cloudflare now queries the running engine at deployment time to verify this. This is the kind of check that feels unnecessary until it isn't.

**Long-context use is the sharp edge.** All 155 affected requests were longer than 681,280 tokens. At typical enterprise usage, very long prompts might be rare, which is why the leak could run for 13 days before being caught externally. If you're not actively testing correctness on long inputs—not just status codes—you might not notice when your rejection logic fails silently.

**The SGLang fix is upstream.** Fixes landed in v0.5.20 through v0.5.22; Cloudflare recommends operators upgrade to v0.5.22 or later. The patch retires rejected requests at the admission queue with the original error, before any allocation or KV transfer occurs.

Cloudflare is also working on page-level integrity tags and formal verification of its KV transfer protocol. Formal verification of distributed memory systems is expensive, but if your disaggregated inference setup is running customer data across trust boundaries, the cost calculus looks different than for a single-tenant cluster.

The broader lesson isn't new—defense in depth, test output correctness not just status codes, don't let a known upstream bug sit in production—but the specific failure surface is unfamiliar to most web engineers. GPU memory pages persisting between requests, disaggregated prefill/decode topologies, admission queues that can silently swallow rejections: this is the new surface area that inference infrastructure teams are learning to reason about.
