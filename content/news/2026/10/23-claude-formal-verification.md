---
title: "A Hundred and Eighty Million Lines of Proof"
date: 2026-10-23T07:00:00+01:00
draft: false
slug: claude-formal-verification-rocq
categories: [research]
tags: [security, formal-methods, anthropic, rocq, coq]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic and Theorem published a large-scale formal verification effort
    where Claude wrote ~180 million lines of Rocq proofs, proving no-crash
    properties for 239 OSS-Fuzz entry points at compiled RISC-V binary level.
    The work found hundreds of bugs—including ones fuzzing had missed for
    years—and forces an honest look at what "formally verified" can and
    can't mean.
---

Formal verification has spent decades as the gold standard nobody could actually afford. Proving a real program correct requires writing proofs that dwarf the program itself in size and complexity, and human proof engineers are expensive and scarce. The usual response has been to either limit scope dramatically (prove a kernel module, prove a cryptographic primitive) or accept that the dream of verified production software is roughly as practical as hand-carving every transistor.

Anthropic and the verification firm [Theorem published a result yesterday](https://www.anthropic.com/research/formal-verification) that puts a different pressure on that assumption. Using Claude, they produced roughly 180 million lines of Rocq proofs (Rocq is the proof checker formerly known as Coq) covering 239 fuzz-test entry points across 123 open-source libraries from Google's [OSS-Fuzz](https://github.com/google/oss-fuzz) program. The property being proved in each case: for any input under 64 MB, the compiled RISC-V binary never crashes. A typical individual proof runs to about 600,000 lines of Rocq and covers a binary with a median of 18,000 RISC-V instructions.

The key thing the work demonstrates is that the bottleneck has flipped. Writing proofs was the hard part. Checking them—running Rocq—is just computation. Claude does the hard part. The spec is generated automatically from the existing fuzz target. Rocq verifies the proof regardless of how many millions of lines it spans.

## What the proofs found

During the verification process, Claude patched hundreds of bugs. Some were the kind fuzz testing reliably surfaces; several were not. A decompressor contained a counter underflow that only triggered on inputs of around 20 MB with a specific memory layout. An image decoder had an out-of-bounds read that needed a 64 MB input and over 2 GB of memory to hit. One library's file-restriction feature wasn't always enforced—no crash involved, just incorrect behavior. The paper also mentions rediscovering [CVE-2026-56132](https://www.cve.org) in libexpat, which requires deeply nested input.

This matters because fuzzing coverage is not uniform. Fuzz testing finds the bugs that are reachable from common inputs; it systematically undershoots on bugs that require unusual sizes, rare call sequences, or specific multi-handle interactions. Formal verification is indifferent to input rarity—a path that needs a 20 MB input with a rare layout is neither harder nor easier to cover in the proof.

## The collaborative efficiency result

The team also ran an experiment where multiple Claude agents shared a workspace and could copy each other's completed proofs. Shared agents used 17% fewer tokens than solo agents and succeeded on some proofs where solo attempts failed, simply because much of each proof covers shared runtime code. When one agent solved a proof for, say, the allocator behavior, others could adapt rather than rediscover.

This is a practically important result. The major cost barrier is token consumption—proofs require hundreds of millions of tokens. If parallelism can cut that through proof reuse, the economics improve faster than per-token cost reductions alone would suggest.

## The honest accounting

None of this means the libraries are bug-free. The proofs hold under a specific set of assumptions: single-threaded execution, static linking, RISC-V architecture, musl libc, a fixed memory budget. Most deployed software runs on x86 or ARM with dynamically linked glibc. The specs themselves might not match intent—a spec generated from a fuzz target says "never crash," but a logic error that produces wrong output without crashing is out of scope. The Rocq checker is trusted, but so is the OCaml compiler it runs on, and the machine running that.

The authors are appropriately candid about this. They note that Claude could in principle "game" the goal by rewriting code to trivially never crash; they penalized and audited such patches, and required proofs of the behavioral delta when patches were made. But the adversarial loop between a model producing proofs and auditors checking patches is itself a research artifact, not a shipping product.

What the work *does* establish is that the cost of writing a formal proof at assembly level, for a real library, is now a matter of tokens and compute rather than person-years. When Opus 5.5 proves roughly half of a small benchmark task set using 40% fewer tokens than Opus 5, that's a rate of improvement that extrapolates well. The gap between "we proved it for RISC-V with musl" and "we proved it for the binary you're actually shipping" is a gap in modeling, not in proof-writing ability.

The group is releasing a subset of the proofs, the RISC-V and OS semantics, and specs for fuzz tests they couldn't prove. The most security-relevant proofs—ones that would reveal unpatched vulnerabilities—are being withheld until fixes ship. That's a reasonable handling of responsible disclosure, and also a sign that the work is finding real bugs in real software rather than exercising a demonstration environment.
