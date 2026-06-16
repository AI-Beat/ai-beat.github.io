---
title: "Memory That Doesn't Help You Think"
date: 2026-06-16T06:12:00+00:00
draft: false
slug: gitofthoughts-agent-memory
categories: [research]
tags: [agents, memory, evaluation, research, open-source]
params:
  author: AI Beat Desk
  summary: >-
    GitOfThoughts stores an LLM agent's reasoning tree as a git repository —
    thoughts as commits, scores as notes, outcomes as tags — which is a neat
    piece of engineering on its own. But the paper's real contribution is the
    negative result buried underneath: none of five memory substrates,
    including their own, reliably improve accuracy on problems that aren't
    near-duplicates of something already seen.
---

Giving an agent "memory" is one of those ideas that sounds obviously good until you try to measure what it actually buys you. Store the agent's past reasoning somewhere — a scratchpad, a vector database, a knowledge graph — and let it retrieve relevant precedent before tackling a new problem. It's the kind of feature that ships because it's intuitive, not because anyone ran the control.

[GitOfThoughts](https://arxiv.org/abs/2606.14470), from Pavan C Shekar, Abhishek H S, and Aswanth Krishnan, is interesting on two levels. The first is the engineering trick in the name: it represents an agent's reasoning tree as an actual git repository. Each scored thought is a commit, scores live as git notes, outcomes become tags, and retrieving relevant history is just `git log` over the agent's own commits. That's a clean way to get diffing, merging, branching, and auditability essentially for free — git already knows how to do all of that, and it's a format every developer can inspect without bespoke tooling. If you wanted to know why an agent converged on a particular answer, or compare two reasoning branches it explored, you'd have actual version-control primitives to do it with instead of grepping through a JSON blob.

The second level is the more important one, and it's the part that distinguishes this from a typical tooling release: the authors used GitOfThoughts as one of five memory substrates — alongside no memory, markdown notes, vector retrieval, and graph-based memory — to test something with actual experimental rigor: two benchmarks, two model scales, and pre-registered replications. The finding is that none of the five formats reliably improved accuracy on novel problems. Not their own git-based approach, not vector retrieval, not graph memory.

The one place memory did help was when the retrieved case was a near-duplicate of the current problem — similarity above roughly 0.8 — at which point accuracy jumps. But that's not transfer of a method or strategy; it's answer lookup. The agent isn't reasoning better because it remembers a similar problem, it's finding the same problem again and copying the answer. Below that similarity threshold, where genuine generalization would have to be doing the work, none of the substrates moved the needle. The only thing that consistently helped across conditions was test-time sampling — just running inference more than once and selecting among attempts — which says something slightly deflating: more compute spent on retrieval architecture didn't beat more compute spent on simply trying again.

The paper apparently includes a retracted result and a refuted hypothesis from the authors' own earlier work, which is a small thing but worth flagging — most papers introducing a new system find a way to make the system look good. Reporting that your own substrate doesn't clear the bar you set, and walking back something you previously believed, is the kind of intellectual housekeeping the field could use more of. It's a useful corrective for anyone building agent products on the assumption that bolting on a memory layer is a free accuracy win: the bar for genuine method transfer, as opposed to retrieval of near-identical past answers, turns out to be a lot higher than the marketing copy for most "agent memory" products implies.
