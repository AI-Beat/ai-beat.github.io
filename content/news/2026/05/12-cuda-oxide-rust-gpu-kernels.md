---
title: "NVIDIA's cuda-oxide Wants GPU Kernels Written in Rust"
date: 2026-05-12T06:08:58+00:00
draft: false
slug: cuda-oxide-rust-gpu-kernels
categories: [tooling]
tags: [cuda, rust, gpu, nvidia, compilers, inference, infrastructure]
params:
  author: AI Beat Desk
  summary: >-
    NVIDIA's NVlabs released cuda-oxide v0.1.0 on May 7, an experimental
    compiler that takes standard Rust and emits NVIDIA PTX directly — no CUDA
    C++, no DSLs, no foreign language bindings. The pipeline goes through a
    custom rustc codegen backend and a Rust-native MLIR-like IR called Pliron.
    Alpha-stage and Linux-only, but it signals where NVIDIA thinks GPU kernel
    development might eventually land.
---

Writing a custom CUDA kernel in 2026 still means writing C++. Not C++ with some GPU extensions bolted on — genuine C++, with all the manual memory management, undefined behavior landmines, and toolchain friction that implies. For most AI workloads, you never touch a kernel directly: PyTorch, JAX, Triton, and a constellation of vendor libraries absorb the low level. But when you need something custom — a novel attention variant, a custom quantization scheme, a fused operator the framework authors didn't anticipate — you're back in CUDA C++ with a compiler that can be difficult to reason about.

[cuda-oxide](https://nvlabs.github.io/cuda-oxide/index.html), released as v0.1.0 by NVIDIA's NVlabs on May 7, is an attempt to change that. It's an experimental compiler that lets you write GPU kernels in Rust — not Rust calling into CUDA via bindings, and not a domain-specific language that happens to look like Rust, but standard Rust that the compiler translates to NVIDIA PTX.

## How It Works

The compilation pipeline is worth understanding because it's genuinely novel. Rust compiles to a mid-level intermediate representation (MIR) before LLVM IR. cuda-oxide intercepts at the MIR stage and routes device-side code through [Pliron](https://github.com/vaivaswatha/pliron), a Rust-native MLIR-like compiler infrastructure, then lowers it through a custom codegen backend (`rustc-codegen-cuda`) that emits PTX. Host and device code share a single source file and are built together with `cargo oxide build`.

The supported SIMT feature set is reasonably complete for serious use: shared memory, warp-level operations, barriers, TMA (tensor memory accelerator), cluster operations, and atomics are all present. Two host-side runtimes are offered — `cuda-core` for explicit resource management and `cuda-async` for composable async operations. In other words, this isn't a toy that can only launch a matrix multiply kernel; it targets the full programming model you'd use for production custom ops.

## What Rust Brings to the GPU

The appeal isn't mainly about safety in the typical Rust sense. GPUs have their own memory model where the ownership invariants are quite different — threads share registers, shared memory has its own lifetime rules, and data races at the warp level are intentional in some patterns. The documentation is explicit that cuda-oxide's safety model has GPU-specific nuances that require careful study.

What Rust does offer is a better ergonomic baseline than C++: a modern type system, useful enums, real closures with captures, monomorphizable generics, and cargo as the build system instead of whatever fragile CMake soup you'd otherwise manage. Kernels in cuda-oxide support generic functions, closures with captured values, and user-defined structs and enums — things that are possible but awkward in CUDA C++.

There's also a practical argument about tooling. Rust's ecosystem — linters, formatters, documentation generators — works on cuda-oxide code because it's still Rust. The GPU-specific parts only matter at compile time. You can write your kernel logic in a module, test the CPU path (where Rust's memory model fully applies), and then compile the device path separately.

## Where It Stands

v0.1.0 is an early alpha. NVIDIA is clear about this: expect bugs, incomplete features, and API breakage. The current constraints are significant: Linux only (tested on Ubuntu 24.04), LLVM 21+ with the NVPTX backend, and nightly Rust. The dependency on nightly is partly structural — cuda-oxide uses compiler internals that stable Rust doesn't expose — so this isn't likely to be resolved soon.

The project sits in NVlabs rather than the main NVIDIA developer organization, which is the appropriate setting: it's research infrastructure, not a supported product. That also means it's under active exploration rather than a commitment to long-term stability.

There's an obvious comparison to [Triton](https://triton-lang.org/), OpenAI's Python-to-PTX compiler, which has become the dominant way to write custom GPU kernels for ML. Triton targets a higher level of abstraction — it manages the SIMT execution model for you, which is exactly what most ML engineers want. cuda-oxide is lower: it exposes the SIMT execution model directly, which gives you more control at the cost of more responsibility. These are complementary rather than competing bets.

The broader pattern here is that GPU kernel development is due for better tooling, and multiple organizations are taking different angles on the problem — Triton from above, cuda-oxide from the systems-language direction, and various MLIR-based compilers targeting specific hardware backends. The question is which abstractions survive contact with the diversity of real ML workloads. v0.1.0 is NVIDIA testing an idea, not shipping a solution. That's exactly what a v0.1.0 should be.
