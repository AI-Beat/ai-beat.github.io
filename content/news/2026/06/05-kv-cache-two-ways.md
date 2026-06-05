---
title: "The KV Cache Is More Compressible Than You Think"
date: 2026-06-05T06:09:49+00:00
draft: false
slug: kv-cache-two-ways
categories: [infrastructure]
tags: [inference, kv-cache, quantization, architecture, efficiency, vllm]
params:
  author: AI Beat Desk
  summary: >-
    Two papers published this week attack the KV cache memory bottleneck from
    opposite directions: one proposes sharing key and value projections at
    training time for a 50% cache reduction with 3.1% perplexity cost, the
    other quantizes stored cache values to 4-bit keys and 2-bit values with no
    calibration required and throughput above FP16. Together they suggest
    the cache is far more compressible than inference engineers typically assume.
---

At inference time, the KV cache is usually the dominant memory pressure. It grows linearly with sequence length and batch size, it lives in GPU memory for the entire request lifetime, and unlike model weights it cannot be quantized once and cached — it is freshly computed per request. For long-context workloads with large batches, the KV cache routinely consumes more memory than the model itself.

Two independent papers published this week approach the same problem from opposite ends.

The first, [Do Transformers Need Three Projections?](https://arxiv.org/abs/2606.04032) (ICML 2026), asks a surprisingly underexplored architecture question: in standard attention, \(\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V\), are all three separate weight matrices \(W_Q\), \(W_K\), \(W_V\) actually necessary? The authors test three projection-sharing strategies systematically — sharing K and V (Q-K=V), sharing Q and K (Q=K-V), and a single shared projection (Q=K=V) — across vision and language tasks up to 1.2B parameters on 10B tokens.

The Q-K=V result is practical: if \(W_K = W_V\), the stored tensors in the KV cache are the outputs of a single shared projection, which halves the cache directly. The perplexity cost in language modeling is 3.1%. Combined with grouped query attention with 4 groups (GQA-4) the reduction reaches 87.5%; combined with multi-query attention (MQA) it reaches 96.9%. The reason Q-K=V works at all is that keys and values naturally occupy similar representational subspaces — the sharing does not fight the geometry of the task. The reason Q=K=V fails is more fundamental: queries and keys serve asymmetric roles (what to look for vs. what to expose), and collapsing them breaks attention directionality in a way that degrades quality significantly.

The second paper, [KVarN](https://arxiv.org/abs/2606.03458) from Huawei, works the quantization side. Assuming you have an existing trained model with standard separate projections — the typical case in production today — how aggressively can you quantize the stored cache at inference time? Their approach combines three techniques: Hadamard rotation along the channel dimension to spread quantization-hostile outliers, iterative variance normalization (Sinkhorn-like) to equalize variance across quantization tiles, and asymmetric precision — 4 bits for keys, 2 bits for values.

The asymmetric bit allocation reflects something real. Key activations feed into the attention score computation, where errors alter which tokens attend to which; value activations feed only into the aggregation step, weighted by already-computed attention scores that are fixed. Values can tolerate heavier compression without the same downstream quality impact. 4+2 rather than uniform 4+4 is a considered tradeoff, not an arbitrary one.

Results on Qwen3-32B: 3-5x more KV cache capacity, throughput above FP16, FP16-level accuracy on MATH500, AIME24, and HumanEval. The [vLLM implementation](https://github.com/huawei-csl/KVarN) requires one flag — `kv_cache_dtype="kvarn_k4v2_g128"` — and no calibration, no model modifications.

The practical near-term story is KVarN: it applies immediately to any existing vLLM deployment and is calibration-free, which matters because calibration datasets add operational complexity that production teams often skip. The QKV paper result requires training from scratch with modified architecture, which means it will surface in the next generation of models — watch for Q-K=V or equivalent descriptions in architecture docs for new base model releases.

What both papers share is a willingness to ask whether the cache is as large as it has to be. The dominant assumption in inference infrastructure has been that KV cache growth is an unavoidable consequence of sequence length and attention, to be managed with hardware (more VRAM) or serving tricks (paged attention, prefix caching). These papers suggest the cache itself has slack that can be removed at the mathematical level, either by not generating separate values to store in the first place, or by storing them at much lower precision than the defaults assume.

Neither result is a free lunch. The 3.1% perplexity hit from Q-K=V is modest but not zero, and the 96.9% reduction combined with MQA involves trading model quality on multiple fronts simultaneously. KVarN's throughput advantage comes with the complexity of the Hadamard+Sinkhorn preprocessing pipeline. But the direction is clearly pointing toward a more compressible cache than the field has assumed, and that has practical implications for how far sequence length and batch size can scale before memory becomes the hard constraint.
