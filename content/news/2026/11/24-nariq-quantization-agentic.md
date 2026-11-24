---
title: "Calibrate for the Task You're Actually Running"
date: 2026-11-24T06:30:00+00:00
draft: false
slug: nariq-quantization-agentic
categories: [inference]
tags: [quantization, nari-labs, qwen, agents, efficiency, local-inference]
params:
  author: AI Beat Desk
  summary: >-
    Nari Labs released NariQ, a sub-3-bit quantization of Qwen3.8-27B that fits
    in 10 GB while matching BF16 on agentic benchmarks. The key insight is mundane
    and probably obvious in retrospect: calibrate the quantization on agent session
    traces, not on web-text perplexity. Standard low-bit quants that look fine on
    perplexity had been failing badly on long-horizon agent tasks.
---

Most quantization research aims for the same target: minimize perplexity loss. Fit the quantized weights as closely as possible to the floating-point weights, as measured on a sample of web text, and declare success. This has worked well for tasks like question answering and summarization. It has not been working as well for coding agents that run for 100,000 tokens, call tools dozens of times, and need to track long contexts without losing the thread.

Nari Labs' [NariQ paper](https://arxiv.org/abs/2611.26157), released last week, is a direct response to that mismatch. Their observation: existing ≤3-bit quantizations of Qwen3.8-27B look fine on AIME math problems but fall significantly behind on [DeepSWE](https://arxiv.org/abs/2611.26157) coding-agent tasks where the BF16 baseline solves 44 of 113 problems. The quantization error that perplexity scores aren't catching is apparently the error that matters most when a model is acting over long horizons.

Their fix is to change the calibration data. Rather than sample web text, NariQ collects the token sequences that the BF16 model itself generates during coding-agent sessions — the actual distribution of completions, tool calls, and partial reasoning chains that an agent produces in practice. Trellis-coded post-training quantization (the core method, adapted from prior work) is then run against this distribution. The intuition is that the quantization error profile you minimize should match the error that actually matters downstream, and in an agent workload the downstream activations look different from random web text.

The results at 3.0 bits per weight are striking: on DeepSWE (83.0% BF16 vs 84.9% NariQ) and Terminal Bench (62.6 vs 63.8), the 3-bit checkpoint sits at or above BF16. At 2.5 bpw, DeepSWE stays essentially flat (84.5%) but Terminal Bench drops a few points (60.2 vs 62.6). AIME and RealWorldQA hold steady across both bit widths. At 2.0 bits the story changes — DeepSWE falls to 75.7% and Terminal Bench to 52.4, clearly behind BF16. That gap between 2.5 and 2.0 bpw seems to be a floor the method can't push through, at least at this model scale.

The file sizes are what motivated people on [HN this morning](https://news.ycombinator.com/item?id=50611402): the 3-bit GGUF is 10.12 GB and the 2.5-bit version is 8.44 GB, against a 53.8 GB BF16 baseline. A 27B model that fits in a consumer GPU allocation while matching the full-precision model on agentic tasks is practically useful in a way that most quantization papers are not.

There is a catch: these files require a [Nari Labs fork of llama.cpp](https://github.com/nari-labs/nari-llama.cpp) because the trellis-coded tensor format isn't in mainline llama.cpp or Ollama yet. That limits adoption for now — most people want to drop a GGUF into whatever runtime they already have. Whether the format lands in upstream llama.cpp will depend on community interest and whether the method continues to look good as more people test it. The paper's benchmarks are from the authors; independent replication takes time.

The broader point NariQ makes is worth internalizing regardless of whether this specific method wins. Post-training quantization is a calibration problem, and calibration data is a design choice that researchers have been treating as a boilerplate step — just grab some C4 or WikiText. When the target workload has a different activation distribution than those datasets, you should not be surprised when the quant underperforms. NariQ's contribution is less the trellis coding and more the demonstration that this matters enough to fix.
