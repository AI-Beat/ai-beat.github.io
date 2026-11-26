---
title: "24 Kilobytes to a Chat Prompt"
date: 2026-11-26T06:11:46+00:00
draft: false
slug: baremetal-qwen
categories: [inference]
tags: [inference, hardware, assembly, qwen, bare-metal, x86]
params:
  author: AI Beat Desk
  summary: >-
    Someone wrote a 24 KB bare-metal x86 OS in assembly that boots directly into
    a Qwen3-0.6B chat session in under 100 milliseconds. No Linux, no C, no libc.
    It's not useful for production, but it makes a clear point about how thin the
    minimum substrate for LLM inference actually is.
---

The [llm.asm project](https://github.com/alesha-pro/llm.asm/tree/main/02-baremetal-os) is exactly what it claims: a bootable x86-64 operating system, written entirely in assembly, whose sole purpose is running Qwen3-0.6B. No Linux underneath. No C runtime. No libc. The kernel itself strips down to 24,252 bytes.

Boot time to a live chat prompt: 74 to 94 milliseconds cold.

The architecture is clean in a way that production stacks obscure. The kernel brings up 64-bit Long Mode, initializes Local APIC across all available cores with an INIT-SIPI-SIPI sequence, sets up a PS/2 keyboard driver, serial port, VGA output, and ACPI poweroff — and that's it. The model (a 614 MiB `.asmbin` file with FP16 scales) arrives as a Multiboot module and gets mapped directly from physical RAM with 2 MiB huge pages, zero-copy. A 449 MiB KV-cache arena is carved from whatever memory is left. When the boot sequence finishes, the system is running inference.

Throughput on a 16-vCPU KVM guest (AMD EPYC 7642) comes out to about 134.9 tokens per second decode, which is not far from what you'd see with a similarly sized model on a comparable Linux setup. The design requires AVX2, FMA, and F16C, so it's effectively targeting server-class x86 hardware from the last decade. The author verifies bit-exact token output against the Phase 1 Linux-based implementation across 1 to 32 vCPUs, which is a reasonable way to establish that eliminating the OS didn't break the math.

None of this is useful in production. You can't run a web server, you can't multiplex users, you can't do anything except talk to Qwen in a boot console. The model is small enough that quantized versions run on a phone.

But the project answers a question worth asking: what is the actual minimum substrate for inference? The answer turns out to be a 24 KB kernel, a CPU with AVX2, and enough RAM for the model and its KV cache. The Linux kernel, the C standard library, the Python interpreter, the CUDA runtime, the inference framework — none of it is load-bearing for the core computation. It's all scaffolding for use cases the model itself doesn't need: filesystem access, networking, process isolation, signal handling, dynamic linking. Strip it out, and inference still runs.

This is a different point than "LLMs are lightweight." Qwen3-0.6B is genuinely small, and the project doesn't suggest you can shrink a Qwen4-Max to 24 KB. The point is structural: the abstraction layers that make serving practical for real workloads add no intrinsic efficiency to the core matrix math. A system designed from scratch for one model and one task can shave them entirely.

The 74 ms cold boot matters mostly as a demonstration. It says that the time between "machine turns on" and "model is answering questions" is bounded below by memory bandwidth and CPU initialization, not operating system overhead. That's a narrow bound, but it's a true one.
