---
title: "Cerebras CS-4: 30× From the Same Silicon"
date: 2026-08-19T06:10:00+00:00
draft: false
slug: cerebras-cs4-power-delivery
categories: [inference]
tags: [hardware, inference, cerebras, wafer-scale]
params:
  author: AI Beat Desk
  summary: >-
    Cerebras unveiled the CS-4 on August 18 without changing its WSE-3 wafer
    at all — the gains come from moving power conversion 100× closer to the
    die and adding a third wafer per rack. The result is a system claiming
    4,400 tokens/sec/user, a number that is hard for GPU clusters to match
    at low batch sizes, along with a 125–135 kW rack TDP that represents the
    real engineering bet.
---

On August 18, Cerebras [announced the CS-4](https://www.cerebras.ai/blog/introducing-cerebras-cs-4): a new rack-scale AI inference system claiming up to 30× faster token delivery than GPU-based alternatives. The headline number is striking. The engineering story behind it is more interesting than the marketing.

## Same wafer, twice as fast

The CS-4 does not introduce a new processor. It uses the same WSE-3 wafer-scale engine that powered the CS-3 — the same 5nm die, the same 44 GB of on-chip SRAM, the same fundamental architecture. What changed is how much power gets to it.

In conventional GPU board designs, power conversion happens at the board level, which means the raw DC current travels some distance before reaching the processor. Cerebras describes moving conversion "100 times closer to the processors" in the CS-4, which almost eliminates resistive losses along the board traces. The practical result: the same wafer can be clocked at roughly twice the frequency, effectively doubling its token throughput.

That one insight — power delivery as the binding constraint, not silicon — explains most of the CS-3-to-CS-4 improvement. The CS-4 also adds a third wafer to the rack (up from two on the CS-3) and doubles off-wafer I/O bandwidth from 1.2 Tb/s to 2.4 Tb/s, cutting wafer-to-wafer network latency to 3 microseconds. Combined, the architecture [claims 4,400 tokens/sec/user](https://convergedigest.com/cerebras-cs-4-wafer-scale-ai-inference/) on production workloads.

## Where this actually wins

The 30× figure deserves context. GPU performance in tokens-per-second is typically measured at large batch sizes, amortizing memory transfer costs across many simultaneous requests. At low batch sizes — the regime where a chatbot serves individual users who each see exactly one token stream — GPU efficiency drops. Cerebras's SRAM-resident model means there is no HBM bandwidth bottleneck at low concurrency: every user gets nearly the full bandwidth of the chip regardless of how many other users are waiting.

Nvidia's Blackwell systems, by SemiAnalysis's [analysis](https://newsletter.semianalysis.com/p/cerebrass-next-generation-cs-4-fast), reach roughly 100–200 tokens/sec/user under realistic chatbot load. CS-4 at 4,400 is genuinely in a different class for this specific workload type.

The tradeoff is that 44 GB of SRAM per wafer is not a lot. For models above a few hundred billion parameters, the CS-4 requires pipeline parallelism across multiple wafers, and the system is explicitly designed for disaggregated inference — pairing with AMD Helios or AWS Trainium for the prefill stage while CS-4 handles decode. That's not a weakness exactly, but it means the CS-4 is an inference accelerator for specific deployment shapes, not a general-purpose cluster replacement.

## The physical bet

The rack TDP is 125–135 kW. That is roughly double the CS-3's aggregate draw and substantially higher than a comparably-sized Blackwell DGX pod. Cerebras is betting that buyers who care deeply about latency-per-user will accept a meaningful power premium to get it. The modular "backpack" design — which separates power and compute into independently installable halves — is partly an answer to the data center integration problem that entails. Customers can stage infrastructure: run power distribution first, then insert wafer modules later without a full system teardown.

First shipments are expected to customers this quarter, with broader availability in Q3.

The CS-4 demonstrates something broader: that inference hardware differentiation increasingly lives in system design — packaging, power delivery, thermal headroom — rather than in raw silicon. The wafer-scale bet Cerebras made years ago has a different risk profile than anyone building around commodity GPU dies, but it's becoming clearer where that bet actually pays off.
