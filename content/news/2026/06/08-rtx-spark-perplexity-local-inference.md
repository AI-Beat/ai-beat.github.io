---
title: "CUDA Comes to Your Laptop"
date: 2026-06-08T06:20:31+00:00
draft: false
slug: cuda-comes-to-your-laptop-rtx-spark-perplexity-hybrid-inference
categories: [infrastructure]
tags: [hardware, nvidia, inference, local-ai, perplexity, edge-compute, on-device]
params:
  author: AI Beat Desk
  summary: >-
    NVIDIA's RTX Spark puts a Blackwell GPU and full CUDA stack inside a laptop SoC
    — enough to run a 120B-parameter model locally with 1M-token context. At roughly
    the same moment, Perplexity shipped a hybrid inference orchestrator that uses a
    compact on-device model to automatically decide which tasks stay local and which
    escalate to the cloud. Together they sketch what a local-AI platform actually
    looks like in hardware and software.
---

For the last few years the standard assumption in production LLM work has been that inference lives in the cloud. The hardware gap was real: Apple Silicon gave you good bandwidth with modest parameter counts, Qualcomm gave you something similar, but if you wanted to run a serious frontier-class model you needed a data center. Two announcements from Computex 2026 are the clearest signal yet that this is changing.

## The hardware layer

[NVIDIA RTX Spark](https://nvidianews.nvidia.com/news/nvidia-microsoft-windows-pcs-agents-rtx-spark) is a two-chiplet system-on-chip for consumer Windows laptops and compact desktops. The GPU half is a Blackwell die with 6,144 CUDA cores and fifth-generation Tensor Cores with FP4 support — roughly in the RTX 5070 class — and the CPU half is a 20-core Arm design co-developed with MediaTek, joined to the GPU via [NVLink-C2C](https://www.nvidia.com/en-us/technologies/nvlink/) at 600 GB/s. Total unified memory: up to 128 GB of LPDDR5X at 300 GB/s bandwidth.

The 300 GB/s number is what matters for inference. Memory bandwidth is the primary bottleneck for large model decode, not compute. A Llama-3.1-70B model needs roughly 140 GB of memory in BF16 and ~35 GB in INT4; the RTX Spark's 128 GB headroom means you can fit a 70B model at half precision or a 120B model quantized, and serve decode at the full 300 GB/s rather than waiting on PCIe transfers from a separate DRAM pool. NVIDIA is citing exactly this: 120B-parameter models with 1M-token context, fully on-device.

What makes this different from previous Arm laptop AI chips is that NVIDIA is shipping the full software stack with it. CUDA, TensorRT, FP4 quantization, OptiX, DLSS — all of it runs natively. Existing tooling built against the NVIDIA ecosystem (vLLM, llama.cpp with CUDA backend, Triton kernels, anything targeting NVIDIA GPUs) ports to RTX Spark without rewriting for a new ISA. That's not a small thing. The reason Apple Silicon has been underutilized for inference work beyond consumer use cases is that the Metal/MLX stack is a parallel universe from PyTorch/CUDA. Qualcomm's QNN SDK has the same problem. NVIDIA entering this market with full CUDA continuity closes that gap.

NVIDIA is calling RTX Spark the foundation for "NVIDIA OpenShell," a secure agent runtime for on-device workloads with policy controls and privacy sandboxing. The concept is that local inference isn't just a cost optimization — it's a trust boundary. Data that doesn't need to leave the device shouldn't leave the device, and the hardware platform enables that.

Devices from Asus, Dell, HP, Lenovo, Microsoft Surface, and MSI are expected this fall. Over 30 laptop designs and roughly 10 compact desktops.

## The software layer

Having a chip that can run large models locally is one problem. Knowing which tasks should actually run locally — and which should be handed off to a frontier model in the cloud — is a different problem, and it's not trivial to get right.

Perplexity's [hybrid local-cloud inference orchestrator](https://www.perplexity.ai/hub/blog/the-data-center-moves-to-your-machine), announced at Computex alongside Intel CEO Lip-Bu Tan, is a direct attempt to solve this in software. The approach is architecturally clean: a compact on-device model acts as a router, classifying each subtask by two axes before dispatching it. First, data sensitivity — does this task involve financial records, medical information, or other material that shouldn't leave the device? If yes, it stays local regardless of compute requirements. Second, compute intensity — is this task simple enough that a local small model can handle it adequately, or does it require a frontier model to get right?

The routing model makes this decision automatically, mid-task, without the user needing to configure anything. A user running an agentic task that involves browsing the web for public information and also summarizing their confidential contract documents gets the former routed to cloud models (more capable, lower latency on complex reasoning) and the latter handled on-device. The system doesn't ask; it classifies and dispatches.

This pattern — a small model as a router for a mixed local-cloud system — is conceptually similar to what LLM routing libraries like [RouteLLM](https://github.com/lm-sys/routellm) do for cost optimization, but Perplexity's framing is specifically about the compute/privacy tradeoff, not just the cost/quality tradeoff. It ships in [Perplexity Computer](https://www.perplexity.ai/computer) in July 2026, confirmed to run on Intel Core Ultra Series 3 and NVIDIA RTX Spark hardware.

## What this looks like as a platform

Neither announcement is fully baked yet — RTX Spark devices ship this fall, and the Perplexity orchestrator comes in July. But the shape of what they're building together is visible: a hardware substrate with enough memory and bandwidth to run large models locally, paired with a software orchestration layer that routes tasks based on sensitivity and complexity, with cloud fallback for tasks that warrant it.

The comparison point most worth making is Apple's Private Cloud Compute, which takes essentially the opposite approach: Apple wants to route AI inference to *their* cloud infrastructure with privacy guarantees, rather than routing to *your* local device. Apple's is a trust-via-cryptographic-attestation model; Perplexity's is a trust-via-not-sending-at-all model. NVIDIA's hardware makes the latter practically viable at useful model sizes.

For engineers building agent systems, the practical implication is that "local model as a fallback" is going to look increasingly like "local model as a first-class option." The 120B-capable, CUDA-native laptop that ships this fall isn't a workstation replacement; it's a new endpoint tier in an inference hierarchy that previously only had edge microcontrollers and cloud datacenters. The middle is filling in.
