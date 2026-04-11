---
title: "The Bug Is Probably in This File"
date: 2026-04-04T19:05:00+02:00
draft: false
slug: kernel-bugs-carlini
tags: [security, agents, linux]
categories: [safety-policy]
params:
  author: AI Beat Desk
  summary: >-
    Nicholas Carlini ran Claude Opus 4.6 over the Linux kernel source one file
    at a time and collected five confirmed CVEs, including a 23-year-old NFSv4
    heap overflow that had survived every prior audit. The human review queue,
    not the AI's discovery rate, is now the bottleneck.
---

At the [un]prompted AI security conference in early March, Anthropic research scientist Nicholas Carlini showed a demo that got uncomfortable quickly. He pointed [Claude Opus 4.6](https://www.anthropic.com/claude) at Linux kernel source files and asked it — framed as a capture-the-flag challenge — to find exploitable vulnerabilities. It found one. Then another. The script looped over every file in the tree so the model wouldn't circle back to the same bug, and by the end Carlini had a queue of several hundred candidate crashes.

[Michael Lynch's write-up](https://mtlynch.io/claude-code-found-linux-vulnerability/) from April 3 gives the clearest account of what the model actually found. The headline bug is in the NFSv4.0 LOCK replay cache in `fs/nfsd/`. When a server denies a lock request, it generates a response and caches it for idempotency. The buffer it allocates is 112 bytes. The NFS protocol permits owner IDs up to 1024 bytes. An attacker controlling two cooperating NFS clients can provoke a response that writes well past the end of that buffer — reading or corrupting kernel memory over the network without any local access. The commit that introduced the flaw landed in March 2003, predating Git by two years. It had survived more than two decades of kernel audits, Coverity scans, and fuzzing campaigns.

That's not an isolated case. Carlini reported five confirmed patches from this exercise: `nfsd` heap overflow, an `io_uring/fdinfo` out-of-bounds read in the SQE_MIXED wrap check, and fixes in `futex`, `ksmbd`, and a signedness issue. Beyond Linux, Claude developed a [working remote root exploit in FreeBSD](https://github.com/califio/publications/tree/main/MADBugs/CVE-2026-4747) from scratch — reportedly the first remote kernel exploit both discovered and weaponized by an AI. Total wall-clock time: around eight hours.

The quote from Carlini that keeps circulating is this: "I have never found one of these in my life before. This is very, very, very hard to do. With these language models, I have a bunch." He is not a novice. He has published extensively on adversarial ML and security. The point is that heap buffer overflows in production C code require holding a lot of implicit protocol knowledge and memory-model reasoning in mind simultaneously — exactly the kind of structured reasoning that LLMs have gotten much better at over the last year.

The methodology itself is almost insulting in its simplicity. There's no symbolic execution, no taint analysis, no fuzzing harness. Carlini wrote a shell script that feeds source files to Claude one by one, tells it the bug is "probably in this file," and asks for an exploitable vulnerability. The CTF framing helps: models apparently produce more focused output when given a game-theoretic goal rather than an open-ended audit request. The only real engineering was the loop to prevent redundant findings.

What makes this qualitatively different from prior AI security work is the current model generation. Carlini compared Claude Opus 4.6 to earlier models in the same family and found substantially fewer confirmed vulnerabilities from the older checkpoints. This matches a pattern visible elsewhere: model capability in open-ended reasoning tasks seems to have crossed some threshold in the last two model generations, and closed-ended benchmarks didn't fully predict it.

The uncomfortable implication is the bottleneck question. Carlini said he has "several hundred crashes" that haven't been validated yet — not because the AI stopped finding bugs, but because a human has to verify each one before it can be reported or patched. In the HN discussion, mtlynch noted a false positive rate "well below 20%" for Claude Opus 4.6, and there's apparently a second LLM pass that filters findings by trying to reproduce the crash, achieving close to 100% precision before the finding reaches a human. Even so, the discovery rate now exceeds the capacity to act on it. That's a structurally new situation. The usual assumption in vulnerability research is that finding bugs is the hard part; triage and remediation are slow but manageable. If the finding step becomes nearly free and the queue grows unboundedly, the responsible disclosure process doesn't have a clear steady state.

There's an obvious dual-use dimension here. The same methodology that surfaces 23-year-old NFS bugs for patching can surface them for exploitation. Carlini works at Anthropic, which has been fairly forward about the dual-use risk in its [Frontier Red Team research](https://www.anthropic.com/research) on autonomous vulnerability discovery. But a conference talk and a patch series are public; the script is not complicated; and the model is commercially available. The window between "researchers demonstrate this is possible" and "adversaries are running it at scale" is shorter every cycle.

A year from now, the question probably won't be whether AI can find kernel bugs. It will be whether the kernel community's review bandwidth has kept up.
