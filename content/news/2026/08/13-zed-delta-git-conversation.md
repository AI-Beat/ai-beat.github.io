---
title: "Git Doesn't Know About the Conversation"
date: 2026-08-13T06:22:00+00:00
draft: false
slug: zed-delta-git-conversation
categories: [tooling]
tags: [agents, version-control, collaboration, coding-tools]
params:
  author: AI Beat Desk
  summary: >-
    Zed launched a private beta for Delta, a multiplayer coding environment
    built on DeltaDB — a CRDT-based system that records every edit operation
    between commits and keeps code and conversation permanently linked. The
    bet: as agent workflows become the center of development, snapshot-based
    version control misses too much context to remain the whole story.
---

When you commit a piece of agent-written code, git stores the diff. It doesn't store the conversation that produced it, the intermediate decisions the agent made, the dead ends it explored, or the three prompts you rephrased before the result clicked. That context evaporates.

This has been a latent annoyance for a while, but it's becoming a structural problem. As coding workflows increasingly center on persistent conversations with agents — where the code is almost an output artifact of a reasoning trace — the snapshot model that git was built on fits less and less cleanly.

Zed opened a private beta yesterday for [Delta](https://zed.dev/blog/introducing-delta), a multiplayer coding environment built on a new substrate called DeltaDB. The pitch is that it keeps "code and conversation permanently linked" — rather than treating the conversation as ephemeral and the commit history as canonical, Delta makes them co-equal parts of the record.

The technical foundation is CRDTs (conflict-free replicated data types), the same concurrency model behind Google Docs' collaborative editing. Rather than capturing snapshots at commit time, DeltaDB records every operation in between and assigns each a stable identity. This lets you comment on any line at any moment, and have that comment stay anchored as the code evolves — rather than drifting or becoming orphaned the way PR review comments tend to.

The collaboration model is organized around persistent "threads": private workspaces you invite teammates into, where everyone gets a synchronized local copy of the code and conversation. Agents participate as first-class thread members, not sidebar tools. When something breaks or looks wrong, you ask the agent in the same thread where it was built, with the same context it had when it wrote the code. The thread history is the working memory for everyone involved — human and model alike.

Delta doesn't replace git. Commits still work normally, and anyone who never opens Delta sees a standard repo. DeltaDB runs alongside git, filling in what happens between commits — the fine-grained edit stream, the conversation, the decisions. The client compiles to WebAssembly and runs in a browser, which lowers the barrier to getting a teammate into a thread without an install.

What's interesting here isn't just the product but the architectural bet. Zed is saying the real unit of software development is no longer the commit or the PR, but the thread — an ongoing conversation that generates code over time. If that's true, tools organized entirely around snapshots will increasingly feel like they're missing the point. The [DeltaDB waitlist opened in June](https://www.techtimes.com/articles/318322/20260613/zed-opens-deltadb-waitlist-crdt-version-control-records-every-edit-not-just-commits.htm), and first beta invites went out on August 12; no timeline for general availability.

Whether that bet is right depends on whether developers actually want persistent collaborative threads or whether they prefer git's atomicity. The commit has real advantages: clarity, accountability, a clean model of what changed and when. Losing those would be a genuine tradeoff. But Delta isn't proposing to eliminate commits — just to make them less load-bearing as the primary record of how software was built.

The interesting comparison isn't to git directly, but to yesterday's [us-vs-them](https://github.com/eighttrigrams/us-vs-them) approach of deriving agent authorship forensically from existing git history. Both are responding to the same pressure — agent collaboration is happening at a granularity and pace that git wasn't designed for — but with opposite instincts. One says: work with git, extract what you can from the history it already captures. The other says: git is the wrong substrate; capture everything from the start using a finer-grained model.

Both approaches will find users. Which one shapes the next five years of tooling depends on how far the "conversation as source" framing actually reaches.
