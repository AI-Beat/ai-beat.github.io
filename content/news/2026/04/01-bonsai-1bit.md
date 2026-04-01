---
title: "One Bit All the Way Down"
date: 2026-04-01T06:15:00+00:00
draft: false
slug: bonsai-1bit-edge-llm
tags: [quantization, edge-ai, open-weights]
params:
  author: AI Beat Desk
  summary: >-
    PrismML launched Bonsai on March 31, claiming the first commercially viable
    true 1-bit LLMs: an 8B model that fits in 1.15 GB and runs at 131 tokens/sec
    on an M4 Pro. The key word is "true" — every layer, including embeddings and
    attention, is 1-bit, not just the weights in isolation.
---

Most quantization research over the past two years has been a negotiation: take a pretrained FP16 model, compress the weights to lower precision, tolerate a small benchmark regression, ship it. The line keeps moving — 8-bit, 4-bit, 2-bit — but the structure stays the same: a floating-point model that's been squeezed after the fact, with the loss in precision distributed unevenly across layers to preserve accuracy where it matters most.

[PrismML's Bonsai](https://prismml.com/news/bonsai-8b), released March 31, claims a different proposition. Their models are trained from scratch in 1-bit precision, with the 1-bit representation applied uniformly across embeddings, attention layers, MLP blocks, and the LM head. No higher-precision exceptions, no mixed-precision fallback for sensitive layers. The result is an 8B-parameter model that weighs 1.15 GB and runs at 131 tokens/second on an M4 Pro.

The math is roughly what you'd expect: 8B parameters at 1 bit each is about 1 GB, and the actual file size reflects that with minimal overhead. By comparison, an 8B model in BF16 is around 16 GB. The claim of 14× compression is therefore not a compression ratio in the post-training sense — it's a measure of how much smaller the model is when trained natively at this precision compared to its floating-point equivalent.

The technical depth here comes from Caltech, specifically from [Babak Hassibi](https://www.ece.caltech.edu/people/hassibi)'s group, which has spent years on the mathematics of neural network quantization. Their published work focuses on maintaining the structure of learned representations under extreme precision constraints — the problem being that at 1-bit, the gradients used during training are discontinuous, and standard backpropagation breaks down. Native 1-bit training requires approximations in the backward pass (typically straight-through estimators) combined with careful initialization and often modified architectures.

The efficiency numbers at inference time are interesting. On an RTX 4090, Bonsai 8B reaches 368 tokens/second. The [Bonsai 1.7B variant](https://huggingface.co/prism-ml/Bonsai-8B-mlx-1bit) reaches 130 tokens/second on an iPhone 17 Pro Max at 0.24 GB. These are the kinds of numbers that make local deployment genuinely practical, not just technically possible in a constrained demo.

PrismML introduces an "intelligence density" metric — benchmark performance per gigabyte — where Bonsai 8B scores 1.06/GB against Qwen3 8B's 0.10/GB. This metric will predictably be criticized: it's designed to make 1-bit models look good, it weights accuracy near the top of the range more heavily, and it doesn't tell you whether the absolute benchmark numbers are competitive in the ways that matter for real tasks. That skepticism is warranted. The more grounded question is whether, for a given task and hardware budget, Bonsai produces outputs you'd actually use. The benchmark parity claim — "competitive with leading 8B models" — needs to be tested across more diverse evaluations before it's taken as settled.

What's not in dispute: the models are available under [Apache 2.0](https://huggingface.co/collections/prism-ml/bonsai), which means you can actually run them. The weights are on Hugging Face with native Apple Silicon and NVIDIA GPU support. At 1.15 GB for a capable-ish 8B model, the barrier to trying it is essentially zero.

The more significant question behind Bonsai is whether native 1-bit training can scale. The company has released 1.7B, 4B, and 8B models, and the 8B is the most interesting test case because it's large enough to show whether 1-bit precision preserves reasoning quality at non-trivial parameter counts. If those results hold as they release larger models, the trajectory becomes meaningful: a 70B-class model at roughly 9 GB would be genuinely different from anything currently available for local deployment. Whether the training math works at that scale is the open question.
