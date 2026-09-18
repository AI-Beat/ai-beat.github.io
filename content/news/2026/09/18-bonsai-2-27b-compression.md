---
title: "The 27B Model That Fits in 5.9 GB"
date: 2026-09-18T06:10:00+00:00
draft: false
slug: bonsai-2-27b-compression
categories: [inference]
tags: [inference, quantization, open-source, hardware]
params:
  author: AI Beat Desk
  summary: >-
    PrismML's Bonsai 2 27B applies ternary quantization to Qwen3.8 27B and arrives
    at a 5.9 GB model that retains 98.2% of aggregate benchmark performance — fast
    enough for interactive use on an RTX GPU or Apple Silicon, Apache 2.0 licensed.
    At this compression ratio and quality, the case for running full-precision locally
    mostly disappears.
---

There's a threshold somewhere in the compression-quality tradeoff where the question stops being "can I afford this quantization?" and becomes "is there any remaining reason not to use it?" [PrismML's Bonsai 2 27B](https://prismml.com/news/bonsai-2-27b), released yesterday, appears to have crossed it.

The model applies ternary quantization to Qwen3.8 27B, constraining weights to \(\{-1, 0, +1\}\) with FP16 group-wise scaling — arriving at 1.76 effective bits per weight. The resulting checkpoint is 5.9 GB, more than nine times smaller than the full-precision version. On aggregate benchmarks covering reasoning, coding, vision, and agentic tasks, the compressed model scores 83.9 overall and retains 98.2% of the baseline's performance. On an RTX 5090 it decodes at 143 tokens per second; on an Apple M5 Max, 46.8 tokens per second. The context window is 262K tokens, unchanged from the base model. License is Apache 2.0.

The speed numbers are the practical part. 143 tokens per second on a single consumer GPU is fast enough that you'd never notice the compression latency. 46.8 on M5 Max is comfortable for interactive use. These aren't the caveated "theoretical peak" numbers often attached to quantization papers — they're end-to-end decode throughput.

Yesterday's [BITCOS post](/news/2026/09/bitcos-ternary-llm-storage/) covered a different angle: Intel researchers found that zeros account for up to 51.5% of ternary model weights, making standard five-trit packing inefficient, and proposed a bitmap-plus-sign-vector format reaching 1.485 bits per weight. BITCOS is a storage and bandwidth optimization; Bonsai 2 is a deployed model you can download and run. They address different layers of the same problem. A Bonsai 2 checkpoint stored in BITCOS format would be smaller still.

The 98.2% retention is the number that matters most here because it quantifies what you're giving up. At 4-bit integer quantization, the quality loss varies by task but is often detectable on reasoning-heavy benchmarks. At 1.76 bits with ternary representation, losing 1.8% of aggregate performance across coding, vision, and reasoning simultaneously is a better outcome than most practitioners expected from sub-2-bit storage. The gap between ternary quantization's theoretical cost and empirical cost has been narrowing steadily, and Bonsai 2 is a clear data point in that direction.

This is also the second Bonsai release, not the first — the original Bonsai 27B appeared roughly two months ago, also a ternary-quantized Qwen derivative. The stated difference with Bonsai 2 is quality: the score on coding, multimodal reasoning, and long-horizon tool use improved materially while the footprint stayed the same at 5.9 GB. That's iteration on compression quality, not just a repackaging.

The broader pattern this fits into: over the past year, the practical question for local AI deployments has shifted from whether a model of this capability can run on consumer hardware to which packaging is most convenient. The answer, increasingly, is ternary-quantized checkpoints under permissive licenses.
