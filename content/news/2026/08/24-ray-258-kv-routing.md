---
title: "Ray Closes the Loop on KV-Aware Routing"
date: 2026-08-24T06:10:00+00:00
draft: false
slug: ray-258-kv-cache-routing
categories: [inference]
tags: [ray, vllm, llm-serving, kv-cache, routing]
params:
  author: AI Beat Desk
  summary: >-
    Ray 2.58.0, released August 23, completes the KV-cache-aware request routing
    work previewed in 2.57: tokenization now happens on the LLMRouter ingress
    replica, tokens are passed out-of-band to the engine, KV lifecycle events
    broadcast to all routers, and CPU-offloaded cache blocks count toward cache
    hit scoring.
---

[Ray 2.58.0](https://github.com/ray-project/ray/releases/tag/ray-2.58.0), released yesterday, finishes a piece of infrastructure that's been in progress since 2.57: KV-cache-aware routing for LLM serving. It's not flashy, but if you're running production inference at any scale, this is the kind of change that quietly removes a significant category of waste.

## What changed

The core shift is that tokenization no longer happens in the engine. In 2.58.0, the `LLMRouter` ingress replica tokenizes the request itself, makes a routing decision based on KV cache state, and then passes the tokens out-of-band to whichever engine replica is selected. The engine skips re-tokenization entirely.

This sounds like a micro-optimization, but it unlocks the routing decision that actually matters: prefix matching. To route a request to the replica that has already cached its prefix — system prompt, few-shot examples, document preamble — you need the tokens *before* you pick the engine. The old architecture where the engine tokenized meant the router was flying blind, routing by queue length or random selection while leaving prefix cache hits to chance.

2.58.0 adds two more pieces that make this coherent. KV lifecycle events (which cache lines were filled, which were evicted) are now broadcast to every ingress replica, so the router's view of each engine's cache state stays consistent. And the router is now aware of CPU-offloaded KV cache blocks — blocks that have been swapped to system memory count toward a replica's cache hit score rather than being invisible. That matters a lot for models that don't fit cleanly in GPU VRAM: if a block is on CPU, it's still cheaper to fetch it there than to recompute it from scratch.

## The routing strategy

[The routing logic](https://docs.ray.io/en/latest/serve/llm/user-guides/prefix-aware-routing.html) is a three-tier fallback: if replicas are balanced (queue lengths within a threshold), route to whichever has the highest prefix match rate. If the system is imbalanced, fall back to Power of Two Choices. If no replica has meaningful overlap (\<10%), route to the least-loaded. This keeps the system from making cache-aware decisions that worsen load distribution under pressure.

The release also bumps to vLLM 0.26.0 and adds dashboards for KV cache offload/reload and SGLang metrics, which are useful if you're trying to understand where your cache budget is going.

## What this is actually about

Prefix caching is one of those optimizations with compounding returns. Most LLM deployments share significant prefix content: every request to the same system prompt benefits if the cached prefix computation is on the replica the request hits. At small scale this barely matters. At medium scale — a few replicas, meaningful traffic — it's the difference between recomputing the system prompt for every request versus almost never recomputing it.

The previous routing situation was that Ray Serve could *maintain* prefix caches per replica but couldn't *steer* traffic based on them. 2.58.0 closes that gap. It's the kind of plumbing that needs to be right before you can reason confidently about your cache hit rates, and it's now right.
