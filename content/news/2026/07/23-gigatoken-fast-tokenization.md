---
title: "Tokenizing at GB/s"
date: 2026-07-23T08:13:00+02:00
draft: false
slug: gigatoken-simd-fast-tokenization
categories: [tooling]
tags: [tokenization, rust, simd, open-source, inference, preprocessing]
params:
  author: AI Beat Desk
  summary: >-
    Marcel Røed released GigaToken, a Rust tokenizer using SIMD and cache-optimized
    byte-pair encoding that runs 500–1,000× faster than HuggingFace's tokenizers
    and up to 681× faster than tiktoken — reaching 24 GB/s on a single CPU.
---

Tokenization is one of those parts of the ML pipeline that people tend to treat as settled. You run the model's vocabulary tokenizer on your input, get some integers back, and move on. The real bottleneck is presumably elsewhere — attention, matrix multiplications, GPU memory bandwidth. Not the text-to-token conversion.

[GigaToken](https://github.com/marcelroed/gigatoken) challenges that assumption fairly directly. Marcel Røed's Rust library benchmarks at 24.53 GB/s on an AMD EPYC CPU for GPT-2 tokenization, versus roughly 24.8 MB/s for HuggingFace's tokenizer on the same workload. That's a factor of roughly a thousand — not a modest improvement, but an order-of-magnitude gap where many engineers assumed there wasn't one. It's also up to 681× faster than OpenAI's tiktoken on the same machines.

The speedup comes from two places. The first is pretokenization. Before byte-pair encoding (BPE) runs, text needs to be split according to the tokenizer's pretokenization regex — for GPT-style tokenizers, splitting on whitespace and punctuation boundaries according to a specific pattern. HuggingFace runs a general-purpose regex engine for this step. GigaToken replaces it with a hand-rolled state machine using SIMD intrinsics — processing 16 or 32 bytes per clock cycle instead of one at a time. This is the same technique that makes high-performance parsers fast in web servers and database engines: recognize that the input is ASCII-range text and exploit the hardware's ability to compare bytes in parallel.

The second optimization is pretoken caching. In real text, the same substrings appear repeatedly — common English words, function names in code, standard punctuation patterns. GigaToken builds an aggressive cache of pretoken-to-token-list mappings so repeated substrings skip the BPE merge step entirely. Since tokenizers are fixed at inference time (the vocabulary doesn't change over the lifetime of a deployment), this cache can be populated once and hit for every subsequent input.

The library is a drop-in replacement for HuggingFace's tokenizers, available via `pip install gigatoken`. The native API gets the full 500–1,000× speedup; the HuggingFace compatibility layer still achieves roughly 200–300× — which is still substantial. One commenter on the Hacker News thread noted that GigaToken "can tokenize the entire internet in under 7 hours on a single machine," which is less a practical claim than a useful way to frame the scale of the improvement.

Where does this actually matter? A few places. Preprocessing pipelines for pretraining or fine-tuning are the obvious case: if you're tokenizing hundreds of billions of tokens from a text corpus, and tokenization is on the critical path, a 1,000× speedup changes a days-long preprocessing run into hours. Streaming inference for low-latency applications is another: time-to-first-token includes tokenizing the input, and at 24 GB/s even long documents tokenize in microseconds. Research loops — fine-tuning runs, evaluation pipelines, dynamic batch construction — also tokenize text repeatedly, and the cumulative overhead accumulates.

Røed is a PhD student at Stanford, and the acknowledgments name Percy Liang, Tatsunori Hashimoto, and Jure Leskovec — suggesting the work emerged from the Stanford NLP group over roughly the past year. He explicitly notes the code is handwritten rather than AI-generated, which is plausible given the SIMD intrinsics and cache-aware BPE implementation; that kind of low-level optimization doesn't lend itself to automated generation.

The broader observation is that ML infrastructure has a tendency to accumulate implementations that are "good enough" and then go unexamined once the GPU cost dominates the budget. GigaToken is a reminder that the non-GPU parts of the pipeline still have room for classic systems engineering — and that when someone actually measures what's there, the improvements can be surprisingly large.
