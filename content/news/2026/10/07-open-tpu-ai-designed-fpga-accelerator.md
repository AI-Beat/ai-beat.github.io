---
title: "The Chip the AI Built"
date: 2026-10-07T08:00:00+02:00
draft: false
slug: open-tpu-ai-designed-fpga-accelerator
categories: [hardware]
tags: [hardware, inference, open-source, agents, fpga]
params:
  author: AI Beat Desk
  summary: >-
    OpenTPU is an open-source FPGA-based AI inference accelerator that was
    designed by AI agents — RTL, ISA, compiler, profiler and all. It runs
    real models (Qwen3, LFM, Gemma 4) on commodity FPGA hardware. The project
    is a data point on whether AI agents can do serious digital hardware design,
    and the answer is: not badly.
---

There is a certain recursiveness to the premise of [OpenTPU](https://github.com/FeSens/openTPU). Developer FeSens gave AI agents a Kintex-7 FPGA, a question — "can you build the hardware that runs your own inference?" — and let them work. The result is a complete inference stack: RTL design, a custom instruction set architecture, a compiler, a simulator, and a profiler, all targeting a Xilinx Kintex-7 xc7k480t on an Inspur card with two DDR3 channels. It is open-source, Apache 2.0, and it runs models.

## What Was Actually Built

The architecture is deliberately minimal. A sequencer issues one instruction per cycle — 8 × 32-bit words each — to four functional units: a DMA for explicit data movement, a matrix unit for int8 weight multiplication, a vector unit for fp32 activations, and a quantizer to convert results back to int8. There is no cache. Every data movement is explicit in the instruction stream, which makes performance characteristics transparent and avoids the design complexity of coherence.

That no-cache choice is worth noting. Classic systolic array designs (the original TPU among them) are built around the same premise: when you are doing highly structured matrix operations, a scratchpad plus DMA is simpler and more predictable than a cache hierarchy. The agents arrived at a sensible design point.

The ISA documentation lives in `docs/isa.md` and the compiler translates standard model weights into that instruction stream. The project supports 18 model configurations: Qwen3, Qwen3.5, LFM2 and LFM2.5, Gemma 4, SmolLM3, and Phi-4-mini, among others.

## The Numbers

On 4-bit quantized weights, decode throughput on the DDR3-1066 DRAM:

- LFM2.5-230M: 85.8 tok/s decode, 335.4 tok/s prefill
- Qwen3-0.6B: 31.3 tok/s decode, 103.4 tok/s prefill
- Qwen3.5-2B: 12.1 tok/s decode, 41.7 tok/s prefill

DRAM utilization during decode sits at 82–85% of peak bandwidth. That figure matters: it means the bottleneck is memory bandwidth, not compute, which is exactly where small-batch LLM decoding lands on every architecture. The agents got the arithmetic right.

These numbers are not going to challenge a datacenter GPU. An FPGA running DDR3 at 1066 MHz is bandwidth-constrained by construction. But 12 tok/s on Qwen3.5-2B is fast enough for interactive local inference on hardware that costs a few hundred dollars, and the primary point of the project is not to win benchmarks.

## The Context

It is worth setting this against two other recent data points on AI and hardware design. In September, [IEEE Spectrum reported](https://spectrum.ieee.org/llms-for-chip-design) on how OpenAI's hardware team used LLMs to take a benchmark from 0.31% to 88.94% of theoretical capacity in 40 hours — expert engineers using AI as a power tool. A week before that, the [EEBench evaluation](https://eebench.org/blog/can-ai-design-circuit-boards-yet/) tested models on 13 analog/digital design tasks using atopile and SPICE simulation: Claude Opus 5 led at 61.6%, with most models failing when component tolerances collapsed nominal-passing designs.

OpenTPU is a different regime from either. There was no expert hardware team directing the process; the agents authored RTL, decided on the ISA, and built the toolchain. It is also not an evaluation — it is a shipped artifact with working benchmarks on real silicon.

Where it sits on the quality curve is hard to say precisely. The design is simple, which is a virtue in hardware (complexity in RTL is where bugs live and where timing fails). But simple also means it lacks features that would matter for production use: no attention-specific accelerators, no pipelining overlap between layers, no prefetch logic. Whether the agents would have produced something more sophisticated given a different prompt or more compute on the design process is an open question.

The 1,361 commits over roughly a month of active development suggest the project went through substantial iteration. That is consistent with what you would expect from an agentic loop: fast-cycle generation and simulation, narrow the ISA down to what actually runs, benchmark, fix, repeat.

The honest read is that AI agents can produce functional digital hardware at the level of a competent first implementation — not state-of-the-art, but not a toy either. The chip runs models. That was the question. The answer is yes.
