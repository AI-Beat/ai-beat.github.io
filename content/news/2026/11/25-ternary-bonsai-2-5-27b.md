---
title: "Five Gigabytes That Can Code"
date: 2026-11-25T06:13:00+00:00
draft: false
slug: ternary-bonsai-2-5-27b
categories: [inference]
tags: [inference, models, local, quantization, agents, open-source]
params:
  author: AI Beat Desk
  summary: >-
    PrismML released Ternary Bonsai 2.5 27B, a ternary-compressed model that
    fits in 5.9 GB while scoring 75.6% on SWE-bench Verified — just 5 points
    below the full-precision Qwen3.8 27B base it's derived from, and 15 points
    above its predecessor. The key gain is in agentic benchmarks: AppWorld
    jumped from 51.2% to 90.5%, and the model now handles multi-step tool use
    with far fewer loops and false starts.
---

PrismML, a compression lab that grew out of Caltech, [released Ternary Bonsai 2.5 27B](https://prismml.com/news/bonsai-2-5-27b) yesterday. The pitch is simple: a model derived from Qwen3.8 27B with weights compressed to three values (-1, 0, or +1), downloaded in one shot as a 5.9 GB file, running at 204 tokens/second on an RTX 4090.

Ternary quantization is not new. The idea of representing network weights in one of three states — negative one, zero, or positive — goes back to BitNet and the flurry of work following it. Bonsai 2.5 sits in a line of models that have been pushing that compression method toward practical quality on agentic tasks specifically.

The benchmark numbers are worth unpacking. SWE-bench Verified: 75.6%. The full-precision Qwen3.8 27B base scores 80.6%, so the compression costs about 5 points. Bonsai 2 (the previous release, same footprint) scored 60.8%, so the update gains 15 points. DeepSWE v1.1 tells a similar story: 27% for Bonsai 2.5, 0% for Bonsai 2. The jump on those two benchmarks looks large because Bonsai 2 genuinely couldn't do the work: it got stuck in loops and failed to recover from tool errors. Bonsai 2.5 was specifically trained to handle multi-step failure — the PrismML page frames the main improvement as "recovering from failed steps and staying on the task."

AppWorld makes that most concrete. AppWorld is a benchmark that runs an agent through realistic multi-step phone app tasks, requiring sustained tool use and backtracking. Bonsai 2 scored 51.2%. Bonsai 2.5 scores 90.5%, very close to the 93.2% of the full-precision base. That's a large gap closed. Terminal Bench 2.1 goes from 52.8% to 60.1% (base: 70.8%), which is a reasonable improvement but doesn't close the gap in the same way.

The one place Bonsai 2.5 scores worse than Bonsai 2 is LiveCodeBench v6: 83.0% vs 83.8%. It's a small regression, and static code generation is a different skill than agentic recovery, so it's plausible the training tradeoffs moved things in that direction.

What's interesting from an engineering standpoint is that the model apparently runs well on Apple Silicon via MLX in addition to the NVIDIA path through llama.cpp. The H100 numbers (291 tokens/s generation with MTP, 3,200 tokens/s prompt processing on a 2,000-token prompt) are the headlining figures, but the 4090 and Mac paths matter more for the actual use case: someone who wants a capable coding agent running entirely on their laptop without sending requests to a cloud provider.

The 5-point SWE-bench gap versus the full-precision model is a reasonable price for the footprint. A 27B model in full BF16 would sit around 54 GB, requiring multi-GPU setups for most consumer hardware. Getting it into 5.9 GB is a 9x reduction. The question for anyone evaluating it is whether that 5-point gap falls on the tasks they care about, and the AppWorld result suggests that for agentic workloads specifically, the compression has gotten good enough that the answer is often no.

PrismML is releasing weights under Apache 2.0, which means you can use, modify, and deploy commercially. If you've been waiting for a local coding agent that fits on hardware you actually own, this is probably the most capable option available today in that footprint class.
