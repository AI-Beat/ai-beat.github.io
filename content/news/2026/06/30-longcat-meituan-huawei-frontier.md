---
title: "Meituan's Trillion-Parameter Model and the Chip Independence Question"
date: 2026-06-30T06:10:00Z
draft: false
slug: longcat-meituan-huawei-frontier
categories: [models]
tags: [models, open-source, china, hardware, moe]
params:
  author: AI Beat Desk
  summary: >-
    Meituan open-sourced LongCat-2.0 today — a 1.6-trillion-parameter MoE with
    a 1M-token context window trained entirely on domestic Huawei Ascend ASICs.
    It is the first plausible demonstration that frontier-scale pre-training is
    achievable without NVIDIA hardware, arriving on the same week that US export
    restrictions on Anthropic's top models remained in partial force.
---

Meituan, the company most people outside China know as "the food delivery app," released [LongCat-2.0](https://longcat.chat/blog/longcat-2.0/) today: a mixture-of-experts model with 1.6 trillion total parameters, 48 billion active per token, and a 1-million-token context window. That spec puts it in roughly the same tier as DeepSeek V4, which is intentional — the comparison is invited, and the difference is in the silicon.

DeepSeek V4-pro trained on NVIDIA hardware and switched to Huawei Ascend chips for inference. LongCat-2.0 used domestic chips for the entire run: pre-training across what Meituan describes as "tens of thousands of AI ASIC superpods," coordinated using Huawei's Collective Communication Library (HCCL). The cluster reportedly ran 50,000–60,000 chips for the full pre-training compute. Whether those are Ascend 910C cards or a newer variant Meituan has not specified, but the HCCL dependency makes the lineage clear.

This is the part that matters for the hardware story. Training a 1.6T-parameter model is not the same problem as serving one. Inference is relatively forgiving: you can tolerate higher latency, route traffic to whatever chips you have, and swap in a faster accelerator later. Pre-training is unforgiving. It requires sustained, high-bandwidth all-reduce collectives across thousands of nodes, near-zero failure tolerance (a single node dying can cascade), and a compiler/runtime stack that keeps utilization high enough to justify the electricity bill. The conventional wisdom until recently was that you needed NVIDIA's interconnect ecosystem — NVLink, InfiniBand, NCCL — to make this tractable at frontier scale. LongCat-2.0 is evidence that Huawei's stack can handle at least the pre-training phase of a model in that tier, which is a meaningful data point even if Meituan's total training loss and final benchmark performance remain unpublished.

The benchmarks, for now, are thin. The [South China Morning Post report](https://www.scmp.com/tech/tech-trends/article/3358854/china-debuts-biggest-ai-model-trained-local-chips-meituan-releases-longcat-20) covers the hardware angle in detail but does not reproduce capability numbers. The Hacker News thread surfaces one informal test — a nuclear-physics question where LongCat-2.0 gave a "well-reasoned but incorrect" answer, ranking third after Gemini Flash and Qwen 3.7 Plus. One data point is noise. Earlier LongCat Flash models (from Meituan's previous release cycle) scored competitively on SWE-Bench Verified at 60.4%, so the lineage is not starting from zero. The full benchmark picture for 2.0 will have to wait for the technical report.

The open-source status is also not fully delivered yet. The weights show "coming soon" on Hugging Face as of this morning. Meituan has a track record of actually releasing its AI models — LongCat Flash and LongCat Flash Thinking are publicly available with technical reports — so this is more likely a staged release than a vaporware announcement. But the models are not in your hands today.

The timing is hard to ignore. The US has spent the past two weeks navigating a partial export restriction on Anthropic's Mythos 5 and Fable 5, with [the most recent ruling](https://www.nhpr.org/2026-06-27/trump-administration-partially-lifts-export-ban-on-anthropics-most-advanced-ai-model) restoring access to roughly 100 vetted organizations while leaving Fable 5 fully blocked. The explicit intent of export controls on NVIDIA chips was to slow frontier AI development outside the US. The implicit theory was that the compute bottleneck would hold. A food-delivery company successfully running a trillion-parameter pre-training run on Huawei hardware is the kind of result that complicates that theory, independent of exactly how good the model turns out to be.

Meituan's AI division is not a scrappy startup. The company has substantial infrastructure, has been investing in AI seriously since at least 2023, and has the engineering talent to adapt a distributed training stack to non-standard hardware. That context matters: this is not a surprising result from a company no one has heard of. But it does extend the existence proof: frontier-scale pre-training without NVIDIA is possible, and more organizations are now doing it.

The weights will presumably arrive in the coming days. When they do, the community evaluation will be rapid — SWE-Bench, LiveCodeBench, and the standard multimodal suite will give a clearer picture of how close the hardware parity extends to capability parity. For now, the interesting thing is not what LongCat-2.0 scores, but what it took to build it.
