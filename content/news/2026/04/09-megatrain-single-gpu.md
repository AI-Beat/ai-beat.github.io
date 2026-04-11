---
title: "One GPU, One Hundred Billion Parameters"
date: 2026-04-09T06:44:33Z
draft: false
slug: megatrain-single-gpu-100b
tags: [training, infrastructure, memory, efficiency]
categories: [infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    MegaTrain, a new paper from Notre Dame and Lehigh, flips the usual assumption
    about GPU training: instead of fitting parameters into GPU memory, it keeps
    everything in CPU RAM and treats the GPU as a transient compute engine.
    The result is full-precision training of 120B-parameter models on a single
    H200, 1.84× faster than DeepSpeed ZeRO-3 on 14B models, and 512K-context
    training on a single GH200.
---

The default mental model for training large models goes something like this: GPU memory is the hard constraint, you need more GPUs to train bigger models, and the engineering problem is about partitioning parameters and gradients across a cluster. [MegaTrain](https://arxiv.org/abs/2604.05091), a paper from Zhengqing Yuan and colleagues at Notre Dame and Lehigh, takes a different starting point.

What if GPU memory is not the right thing to optimize for? An NVIDIA H200 has 141 GB of HBM. A modern high-end workstation with the same H200 in a PCIe slot has 1.5 TB of host (CPU) RAM. The ratio is roughly 10:1 in favor of the system. At 12 bytes per parameter (BF16 weights + BF16 gradients + FP32 Adam moments), a 120B-parameter model needs about 1.44 TB of storage — more than the H200 can hold, but comfortably within what a single-node workstation already has.

The insight driving MegaTrain is simple: store all parameters and optimizer states in host memory, and stream them to the GPU layer by layer for computation. The GPU becomes a "transient compute engine" rather than a warehouse. For each layer, weights flow in, gradients flow out, and the GPU never needs to hold more than one layer's worth of state at once.

The hard part is the bandwidth gap between CPU and GPU memory. Standard PCIe 5.0 x16 delivers around 64 GB/s host-to-device — roughly 75× slower than the H200's HBM bandwidth of 4.9 TB/s. Naively streaming one layer at a time would leave the GPU starved, waiting for data. The GH200 (Grace Hopper Superchip) is a different story: its NVLink-C2C interconnect between the ARM CPU and the GPU runs at ~900 GB/s, which is why the most impressive numbers in the paper come from that configuration. Either way, MegaTrain's solution is a three-stream pipeline: one CUDA stream prefetching layer \(N+1\) from host memory, one stream computing layer \(N\), and one stream offloading gradients from layer \(N-1\). The three operations overlap in time, hiding most of the data-movement latency regardless of which interconnect you're on.

Two implementation details make this work well. First, rather than maintaining a persistent autograd graph (which requires storing all intermediate activations and keeping track of the full computation DAG), the system uses what it calls "stateless layer templates" — weight-agnostic forward and backward implementations that bind to whatever weights stream in. This eliminates the metadata that would otherwise accumulate across a deep network. Second, all per-layer state — BF16 weights, BF16 gradients, FP32 Adam moments — is packed into a single contiguous block aligned to 4KB page boundaries, which lets the DMA engine transfer it in a single burst rather than chasing pointers across fragmented host memory.

The benchmarks are competitive. On a single GH200, MegaTrain trains a 14B model at 264 TFLOPS against DeepSpeed ZeRO-3's 143 TFLOPS — a 1.84× speedup. The improvement is even larger at 7B on an A100: 128 TFLOPS vs. 36 TFLOPS (3.56×). ZeRO-3's overhead comes from inter-process communication and redundant parameter staging across its multiple parallel ranks; MegaTrain avoids this by running single-process. At 32B and beyond, ZeRO-3 simply runs out of GPU memory on a single node; MegaTrain keeps going, training 120B on an H200 system with 1.5 TB of host RAM — which the model needs almost all of, since BF16 weights + BF16 gradients + FP32 Adam moments comes to 12 bytes per parameter, or roughly 1.44 TB for 120B params.

The long-context case is interesting in its own right. Training a 7B model at 512K sequence length on a single GH200 is possible with MegaTrain — the paper achieves roughly 400 TFLOPS there via chunked MLP execution — and the memory usage grows approximately linearly with model size rather than the near-exponential growth the baselines show due to activation storage and fragmentation.

The obvious question is when single-GPU training is actually useful. For large-scale pretraining, it isn't — you'd still want thousands of GPUs in parallel. But there are genuinely valuable settings where you have one good GPU and need to fine-tune something large, or you're doing research on a 100B+ model and can tolerate slower throughput in exchange for simpler infrastructure. Long-context fine-tuning is a particular sweet spot: a single GH200 can now handle context lengths that would otherwise require multi-GPU setups for baseline models.

The [code is on GitHub](https://github.com/DLYuanGod/MegaTrain) under an open license. The paper was submitted April 6 and picked up traction on Hacker News shortly after. It's not a systems achievement that will change how frontier labs train their next models — those already have thousands of accelerators — but it does meaningfully expand what researchers with access to a single high-end GPU can do on their own.
