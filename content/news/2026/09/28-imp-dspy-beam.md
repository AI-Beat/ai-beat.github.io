---
title: "DSPy Comes to OTP"
date: 2026-09-28T08:12:19+02:00
draft: false
slug: imp-dspy-beam
categories: [tools]
tags: [tools, elixir, dspy, frameworks, agents, open-source]
params:
  author: AI Beat Desk
  summary: >-
    Imp 0.5.0 ports DSPy to Elixir and the BEAM runtime, bringing typed LLM
    signatures, multiple prompt optimizers, and agent loops backed by OTP
    supervision. It's early and experimental, but the combination of DSPy's
    systematic optimization approach and the actor model's fault tolerance is
    a genuinely interesting fit for production agentic systems.
---

[DSPy](https://github.com/stanfordnlp/dspy) made a bet that treating prompts as hand-crafted strings is the wrong level of abstraction. Instead, you declare typed signatures — "this step takes a question and returns an answer with reasoning" — compose them into modules, and run optimizers that automatically find good prompts and few-shot examples by testing against labeled data. The idea is to move from prompt engineering to program optimization.

That bet has been paying off for Python users for a couple of years. Now [Imp 0.5.0](https://github.com/deepfates/imp) ports the framework to Elixir and the BEAM, and it's the first release on Hex, Elixir's package manager.

The surface looks familiar if you know DSPy: you define signatures with typed inputs and outputs, compose them into programs using modules, and run optimizers to improve performance. Imp ships GEPA (which rewrites instructions through reflection), BootstrapFewShot, MIPROv2, and SIMBA. It also includes agent loops and retrieval, and has native support for MCP servers and ACP clients — which means it can call Claude Code, Codex, or similar tools directly from within optimized programs.

The interesting part is what changes when you run this on the BEAM instead of Python.

OTP's supervision model means that agent processes aren't just function calls that either return or raise an exception. Each agent runs as a supervised process in an OTP supervision tree. If a process crashes — network failure, malformed model output, timeout — the supervisor restarts it according to a defined policy. This is the kind of reliability story that matters when you're running hundreds of concurrent agents: individual failures are contained and recoverable rather than cascading.

The BEAM's concurrency model also fits naturally. LLM calls are slow and IO-bound; Elixir's lightweight processes and the actor model handle this well. Running BootstrapFewShot over a hundred examples means a hundred LLM calls in parallel, which BEAM handles without threads or async machinery you have to manage explicitly.

That said, 0.5.0 is described as experimental, and the README is honest about it. The dependency list includes C/C++ compiler requirements for native extensions, which adds friction for Elixir projects that don't already have that in their build environment. DSPy's Python ecosystem — all the existing optimizers, community modules, and integrations — doesn't come along for free.

The question Imp implicitly raises is whether there's enough of an Elixir ML ecosystem to make it worth planting this flag. Phoenix and Livebook are used for ML-adjacent work, Nx provides numerical computation, Bumblebee handles model inference. The ecosystem exists; it's just considerably smaller than Python's. Imp adds systematic LLM program optimization to that toolkit.

For teams already running production Elixir services who want to add LLM functionality, the path of least resistance has been Python sidecars or HTTP APIs. Imp offers an alternative: stay in Elixir, keep OTP's fault tolerance properties, and get DSPy's programming model rather than hand-rolled prompt strings. Whether 0.5.0 is mature enough to actually use in production is a separate question — but the fit between DSPy's abstractions and OTP's runtime model is clear enough to be worth following.
