---
title: "The Agent That Rewrote Itself"
date: 2026-10-10T06:11:06+00:00
draft: false
slug: prime-agent-rust-rewrite
categories: [tools]
tags: [agents, rust, software-engineering, prime-intellect]
params:
  author: AI Beat Desk
  summary: >-
    Prime Intellect rewrote Prime Agent — their open-source coding agent harness, 300k downloads, 8 trillion tokens — from TypeScript to Rust using ~2,000 AI agents across 10,000+ sandboxes. The result is 13x faster startup, 5.7x less memory, and a nine-crate architecture with session isolation. A practical test of whether agent fleets can handle serious infrastructure work, not just toy tasks.
---

Prime Intellect published a [detailed writeup](https://www.primeintellect.ai/blog/prime-agent-rust) this week of rewriting Prime Agent — their open-source coding agent harness, 300,000 downloads and 8 trillion tokens served since August — from TypeScript to Rust. The rewrite took about two weeks and used approximately 2,000 AI agents across more than 10,000 sandboxes. The humans mostly set up the verification harness and made architecture decisions.

It's a good engineering case study, and also a slightly eerie one.

The TypeScript version had the standard problems: dynamic types that vanish at runtime, unchecked exceptions, CPU-bound work sharing an event loop with keyboard input, a garbage collector with inconvenient timing. The Rust port addresses all of that in the expected ways: exhaustive enums, compile-time ownership, separate threads per session, explicit allocation. None of this is novel — the community has written about all of it many times. What's different is who did the work.

The rewrite process used a root orchestrator that split the codebase into tasks ordered by the dependency graph and wrote no product code itself. Each task moved through four roles: a planner that defined the feature and the TypeScript behavior to match; an implementer that wrote Rust in an isolated worktree; an adversarial reviewer running on a different model in a separate context; and a verifier that compiled and ran parity tests in a fresh sandbox. Failed reviews went back to the implementer. Parity was checked at four layers: the terminal UI, the harness, the daemon protocol, and features.

The result is a nine-crate architecture with a one-way dependency graph enforced by Cargo. The largest file shrank from about 15,000 lines to about 2,500. Sessions now run in isolated worker processes under a supervisor — one crash doesn't cascade. Models and MCP server lists are fetched at runtime, so new integrations can ship without a Prime Agent release.

The performance numbers are meaningful for a daemon that runs in the background while you type:

- Cold startup: 55.8ms vs 737.8ms (TypeScript)
- Warm startup: 42.4ms vs 549.6ms
- Memory at startup (RSS): 106MB vs 607MB
- Installed size: 59.6MB vs 172.1MB

A 13x startup improvement matters for interactive tooling. 5.7x less memory matters if you run multiple sessions. A separate hillclimbing loop merged 69 performance changes on top of the functional port.

The meta-point: Prime Intellect used their agent infrastructure to rebuild their agent infrastructure. That's not just a tidy story — it's a practical test of whether AI agents can handle a bounded software engineering task with clear success criteria and adversarial review built in. The answer, based on these results, is that they can, at least in this size range and for a TypeScript-to-Rust port.

The architecture needed human judgment. The code mostly didn't. That's a more specific and more credible claim than "agents can design systems." Given a clear task, explicit verification criteria, and a reviewer running on a different model to catch failures, a fleet of agents produced maintainable Rust that beats its predecessor on every metric that matters for an interactive daemon. Whether that generalizes beyond well-scoped rewrites is still an open question, but it's a better data point than most.
