---
title: "Kimi K3 on a MacBook, via Four SSDs"
date: 2026-09-09T06:11:33+00:00
draft: false
slug: deltafin-kimi-k3-ssd-streaming
categories: [inference]
tags: [inference, local-models, hardware, open-source, moe]
params:
  author: AI Beat Desk
  summary: >-
    Deltafin, a Rust binary for consumer inference, now runs the full
    unmodified Kimi K3 2.8-trillion-parameter MoE model on an Apple Silicon
    MacBook at around 1 token per second, streaming 1.45TB of expert weights
    from four external SSDs. It took six weeks to go from 0.014 tok/s to 1.0.
---

[Kimi K3](https://github.com/MoonshotAI/Kimi-K3) is a 2.8-trillion-parameter mixture-of-experts model released by Moonshot AI. It activates 32B parameters per token. Running it locally is not, on any reasonable accounting, a sensible thing to try. The weights alone don't fit in any consumer GPU configuration. The naive inference path would require something like 140GB of memory bandwidth per token. It is not designed for edge deployment.

[Deltafin](https://github.com/argonautlabsai/deltafin), a project built on the original gavamedia/deltafin codebase, has gotten it to run at approximately 1.0 token per second on an M5 Max MacBook Pro with 128GB RAM — by streaming 1.45TB of expert weights from four external SSDs.

The engineering here is interesting. In a MoE model, each token activates a sparse subset of "expert" weight blocks. If you know which experts will be needed, you can prefetch them from storage before the compute step requires them. Deltafin builds on this by:

1. Using the router predictions from earlier layers to schedule expert prefetches from SSDs before the activated layer is reached
2. Striping the expert weights across multiple drives to maximize read bandwidth
3. Keeping the attention and non-expert weights resident in the 128GB of unified memory

The SSD scaling characteristics the project reports are revealing. One drive gets you to about 52% of four-drive speed; two drives reach 73%; three drives reach 90%. This sub-linear scaling (you'd expect 25%, 50%, 75%, 100% if drives were perfectly independent) reflects the overhead of coordinating prefetches and the fact that some expert weights are hotter than others and create read contention.

At 1.0 tok/s, you're not doing interactive chat with K3. You're running something that takes tens of seconds to produce a paragraph. But that's not necessarily the point. The benchmarks the project tracks tell a different story: on July 27, Deltafin was doing 0.0141 tokens per second. By September 8, it's at 1.0 tok/s. That's a 70x improvement in six weeks, driven entirely by software — smarter prefetching, better scheduler, lower overhead in the Rust binary. The model hasn't changed; the infrastructure around it has.

"Full, never-pruned" is a deliberate design choice in Deltafin. All 16 experts are used for every token; smaller draft models can propose candidates, but K3 must validate each one. The developers are specifically not building a degraded local experience — they're building a slow but complete one.

There are a few things worth noting about where this sits in the larger picture. The M5 Max with 128GB unified memory, plus four external SSDs with a fast hub, is expensive consumer hardware — the kind of rig someone running a serious local AI setup might have, not a commodity machine. The 1.45TB of SSD storage for the expert weights alone is a significant ask. And 1 tok/s is slow enough that you'd only reach for K3 locally for tasks where quality matters more than speed and privacy matters more than cost.

But the trajectory from 0.014 to 1.0 tok/s in six weeks, without any model changes, is a concrete demonstration of what software optimization can do against a fixed hardware budget. The ceiling for this kind of approach isn't clear yet. Whether it gets to 5 tok/s, or 10, depends on how much more is left in the prefetching strategy and the SSD throughput envelope.

"Local AI" has meant different things at different moments in the last few years — small quantized models on laptops, then mid-tier models on high-end workstations. Deltafin is making a case that it can also mean the actual frontier, at a cost in speed and complexity that might be worth paying for the right use case.
