---
title: "LLMs Know the Raft Paper. They Don't Know Etcd."
date: 2026-05-09T06:12:02+00:00
draft: false
slug: llms-tla-plus-sysmobench
categories: [research]
tags: [formal-methods, benchmark, distributed-systems, evaluation]
params:
  author: AI Beat Desk
  summary: >-
    SysMoBench, a new benchmark from the Specula team, tests whether LLMs can
    produce TLA+ formal specifications that accurately model the behavior of
    real distributed system implementations. They score near-perfect on syntax
    and only ~46% on conformance and ~41% on invariant checking — because they
    model the algorithm as described in papers, not as implemented in code.
---

Formal specification of distributed systems is tedious, expert-requiring work, and LLMs are very good at writing TLA+ syntax. The obvious question is whether these two facts combine into something useful — whether you can ask a model to produce a formal spec of, say, your Raft implementation, and get something that actually models what your system does.

A new paper from the Specula team on the [ACM SIGOPS blog](https://www.sigops.org/2026/can-llms-model-real-world-systems-in-tla/) gives a careful empirical answer: not really, and for an interesting reason.

## The Benchmark

SysMoBench evaluates generated specs in four phases: syntax checking, state reachability, conformance (do the model's traces match traces the real system actually produces?), and invariant checking (do properties that hold in the real system hold in the spec?). The benchmark reports per-action granularity rather than aggregate scores, which makes it possible to see exactly where a spec diverges from reality.

The results across leading LLMs:
- Syntax: near-perfect (>90%)
- Conformance: ~46% on average
- Invariant checking: ~41% on average

The models produce code that parses and typechecks. They just don't produce code that models the right thing.

## The Core Failure Mode

The example the paper uses is instructive. When asked to model Etcd's Raft implementation, Claude produces a spec that accurately reflects what the Raft paper says Raft does — but Etcd's actual Raft implementation makes specific choices that diverge from the paper. Multi-step operations that the paper describes abstractly are fused into atomic operations in Etcd's code. Set semantics get used where Etcd uses map overwrites. These aren't bugs in Etcd; they're implementation decisions. But a spec that models the paper rather than the implementation will miss them.

The failure mode has a name: the models are *reciting textbooks*. Their training data contains the Raft paper (and probably many explanations of Raft). It almost certainly doesn't contain a complete, structured representation of "here are the ways Etcd's Raft implementation specifically differs from the paper and why." So asked to specify Etcd's Raft, they produce specification of Raft as they know it, which is the paper version.

This is different from the generic "LLMs hallucinate" observation. The models aren't making things up. They're accurately representing something real — the algorithm as designed — and that's precisely the problem. Real-world systems deviate from design documents in ways that matter for specification.

## Agents Do Better

One result worth noting: agentic tools like Specula itself (which can read the actual codebase, run the system, and observe traces) achieve full conformance and invariant scores on current benchmark tasks. This isn't surprising in principle — if the agent can observe what the system actually does and iterate on the spec accordingly, it can converge on an accurate model — but it's a useful existence proof that the task is tractable with the right setup.

It also points to where the gap is: not in LLMs' inability to write TLA+, but in their ability to form a grounded model of a specific implementation from the artifacts a human practitioner would use (code, commit history, integration tests). Bridging that gap is an information retrieval and reasoning problem, and agentic scaffolding that can run the system and observe its behavior is currently the most direct path.

## What This Means for Formal Methods + AI

There's been real enthusiasm about LLMs potentially democratizing formal methods — lowering the barrier to writing specs for systems that currently go unspecified because the expertise required is hard to find. SysMoBench's results are a useful corrective: the barrier to *syntactically valid* specs is already low; the barrier to *behaviorally accurate* specs remains high, and that's the one that actually matters for verification.

For anyone trying to apply LLMs to formal verification in production, the practical implication is clear. You can use an LLM to draft a spec, but you cannot trust that it models your implementation rather than your implementation's sources. Validation against actual system traces — exactly the kind of setup the agentic tools use — is not optional.

The deeper issue is epistemological. LLMs compress what people write about systems, not what systems do. For domains where the written description and the actual behavior closely align, this works well. For distributed systems, where implementation choices quietly diverge from the specification for performance, correctness, or operational reasons, the gap is systematically large enough to matter.
