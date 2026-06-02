---
title: "MiniMax M3 and the Cost of Long Context"
date: 2026-06-02T06:13:00+00:00
draft: false
slug: minimax-m3-sparse-attention
categories: [models]
tags: [models, long-context, attention, minimax, open-weights]
params:
  author: AI Beat Desk
  summary: >-
    MiniMax M3 launches with a sparse attention mechanism that cuts per-token
    compute at 1M tokens to one-twentieth of its predecessor. The architecture
    is genuinely interesting; the benchmarks require scrutiny; the license
    is almost certainly not what the word "open-weight" implies.
---

The hard part of supporting a 1-million-token context window is not storing the tokens. It is computing attention across them. Standard scaled dot-product attention is quadratic in sequence length: double the context, quadruple the compute. At 1M tokens that number becomes impractical unless you do something about it.

[MiniMax M3](https://www.minimax.io/blog/minimax-m3), released June 1, 2026, is built around an architecture called MiniMax Sparse Attention (MSA) that tries to do something about it. The published mechanism partitions key-value pairs into blocks, then uses what MiniMax calls a "KV outer gather Q" access pattern — KV blocks serve as the outer loop aggregating matching queries, ensuring each block loads only once with contiguous memory access. That is more precise than what the competing DSA and MoBA designs do, and reportedly more than 4× faster than open-source sparse attention implementations like Flash-Sparse-Attention. The key-value pairs are uncompressed; the sparsity is in which blocks get computed, not in a lossy representation of the values. The result is a claimed 1/20th per-token compute at 1M context versus the M2 series, with 9× faster prefill and 15× faster decoding.

Those are large multipliers. On a practical level, MiniMax says M3 processes 1M-token inputs at around 100 tokens per second output — roughly 3× faster than Claude Opus 4.7. The guaranteed floor is 512K tokens of high-fidelity performance, with the full 1M window available but with unspecified degradation characteristics above the midpoint. For agent workflows that repeatedly hit tool boundaries and re-inject large context, the speed difference at scale is meaningful.

The benchmark headline is 59% on SWE-Bench Pro. That puts M3 ahead of GPT-5.5 and Gemini 3.1 Pro on that benchmark, and close to Claude Opus 4.7. On BrowseComp — a benchmark testing web research capability across many hops — M3 scores 83.5, above Opus 4.7's 79.3. Terminal-Bench 2.1 comes in at 66%, and MCP Atlas (tool-use) at 74.2%.

The benchmarks deserve a caveat MiniMax itself acknowledges: several results were obtained on MiniMax's own infrastructure with their own agent scaffolding. Self-reported numbers on scaffolded agentic benchmarks are notoriously hard to compare across providers because scaffold quality matters substantially. The core SWE-Bench Pro number is likely credible — it is a well-defined benchmark — but the agentic numbers should be treated as indicative until third parties reproduce them.

M3 also ships with native multimodal capabilities: images and video, not just text. That is genuinely new for MiniMax models (M2.7 was text-only), and combined with the 1M context and the coding performance, it positions M3 as a model targeting complex, long-running agent tasks rather than simple chat.

The open-weight situation is familiar to anyone who followed M2.7. MiniMax describes M3 as open-weight, with weights and a technical report due on Hugging Face around June 10. But M2.7 launched under a "Modified MIT" license that restricted commercial deployment without written authorization from MiniMax. If M3 follows the same pattern — weights downloadable, commercial use negotiated separately — then "open-weight" describes the artifact, not the freedom to deploy it. Worth confirming before committing infrastructure decisions to it.

Pricing on the API is $0.30 per million input tokens, dropping to around $0.06 blended with caching at realistic cache hit rates. At those prices M3 sits well below Claude Opus 4.7 on cost while claiming comparable performance on most of the benchmarks it publishes. The 1M context at that price point, if the quality holds, changes the economics of long-context pipelines significantly.

The [MiniMax M2.7 post](/news/2026/04/minimax-m27-self-evolving/) from April noted that the interesting part was the process (a model autonomously improving its own training scaffold), not just the numbers. With M3 the interesting part is the architecture: sparse attention is not a new idea, but the specific block-partitioned, memory-access-optimized design at this scale, with open weights pending, gives researchers something to study and practitioners something to run. Whether the published weights match the reported benchmarks will be clear within a week or two.
