---
title: "AMD's Inference Edge Was Two Bug Fixes Away"
date: 2026-08-02T06:30:00+00:00
draft: false
slug: kimi-k3-amd-mi355x-inference
categories: [inference]
tags: [amd, rocm, kimi-k3, hardware, inference]
params:
  author: AI Beat Desk
  summary: >-
    Wafer.ai's benchmarks of Kimi K3 on AMD MI355X tell a story that goes beyond the numbers: the hardware was capable all along, but two ROCm bugs were blocking it. Fixing them yields 48 tok/s per GPU-dollar, against 7 for B200 — a gap that challenges the assumption that NVIDIA owns frontier model inference.
---

[Wafer.ai's July 31 writeup](https://wafer.ai/blog/kimi-k3-mi355x) on running Kimi K3 across AMD MI355X and NVIDIA Blackwell hardware is worth reading not for the conclusion — AMD wins on cost — but for the two paragraphs buried in the middle that explain why AMD wasn't already winning.

The MI355X has 288 GB of HBM3E memory, enough to fit all of Kimi K3's 2.8 trillion parameters in a single 8-GPU node at TP8. The B200 needs 16 GPUs (TP16, spanning two nodes) to serve the model, adding network hops that hurt latency. The hardware advantage is obvious on paper. In practice, nobody was deploying Kimi K3 on MI355X at scale because the ROCm stack had two bugs that stopped it from working properly.

Bug one: sglang's ROCm sampling branch was missing `top_k_renorm_prob`, a function used during speculative decoding. The fix was a single PyTorch call. Bug two: the AITER MLA prefill kernel expected attention head counts of 4, 8, or multiples of 16. Kimi K3 at TP8 produces 12 heads per rank. The fix was zero-padding to 16 and slicing back to 12 afterward. Neither fix required kernel rewriting; both were caught by actually running the deployment and reading the error messages.

With those patches in place, the numbers look like this:

| | MI355X (TP8) | B200 (TP16) | B300 (TP8) |
|---|---|---|---|
| Decode tok/s/stream | 118 | 90 | 172 |
| Peak aggregate tok/s | 952 | 498 | 1,568 |
| Performance per GPU-hr | 48 tok/s | 7 tok/s | 33 tok/s |

GPU pricing: MI355X at $2.50/hr, B200 at $4.25/hr, B300 at $6.00/hr. The B300 is the fastest in raw throughput; the MI355X is the fastest per dollar by a factor that's hard to explain away. At 6.8× the cost-efficiency of B200 before speculative decoding, and 2.2× single-stream improvement after adding [RadixArk's DSpark draft model](https://github.com/radixark/kimi-k3-dsp), the economics shift considerably.

[AMD's own day-0 deployment guide](https://www.amd.com/en/developer/resources/technical-articles/2026/kimi-k3-on-amd-instinct-gpus.html) for Kimi K3 on MI355X validates the setup but doesn't tell the same story as the wafer.ai writeup. AMD's numbers are accurate but sanitized. The wafer.ai post is more useful because it describes what actually went wrong and how to fix it — the kind of operational detail that determines whether anyone actually uses the hardware.

The underlying story here is about ecosystem friction. AMD's hardware has been genuinely competitive on memory capacity and raw FLOPS for a while, but ROCm has historically been the gap: missing kernels, incomplete operator support, subtle numerics differences that only surface in production workloads. Progress is real but uneven. The two bugs wafer.ai fixed are trivial in retrospect. The fact that they were blocking production deployments for a top-tier model suggests there are more like them scattered through the stack, waiting for someone to run the right workload.

The CUDA moat isn't primarily about hardware anymore; it's about the accumulated ecosystem of kernels, profiling tools, and operator libraries that have been tuned by a decade of deployment experience. AMD is closing the gap faster than it was two years ago, and [ROCm.ai](https://www.amd.com/en/blogs/2026/rocm-ai-the-ai-native-developer-experience-for-building.html) represents a serious institutional push to make that tuning more automated and systematic. But "SOTA on AMD is imminent," as wafer.ai puts it, has been a prediction for a while. The MI355X numbers are the best evidence yet that imminent might now mean this quarter.

For anyone serving frontier-scale MoE models and paying attention to inference costs, it's worth running the comparison. The hardware is there. The fixes are documented. The math on $/token is hard to ignore.
