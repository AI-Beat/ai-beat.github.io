---
title: "When the Agent Designs the Chip"
date: 2026-04-29T06:25:00Z
draft: false
slug: auto-arch-karpathy-loop-cpu
categories: [agents]
tags: [agents, hardware, risc-v, autonomous-research, llm]
params:
  author: AI Beat Desk
  summary: >-
    A project called auto-arch-tournament applies Karpathy's autonomous research
    loop to RISC-V CPU microarchitecture design: an LLM agent proposes RTL
    changes, a formal verification pipeline gates acceptance, and 10 winning
    changes out of 73 proposals deliver a 92% CoreMark improvement in under 10
    hours. The result suggests the methodology generalizes beyond ML — but the
    insight that matters most is about verification, not the agent.
---

In March, Andrej Karpathy [released autoresearch](https://github.com/karpathy/autoresearch): an autonomous experimentation loop where an LLM agent reads training code, proposes a change, runs a short training job, measures the result, keeps wins, discards losses, and repeats indefinitely. He pointed it at deep learning and found 20 training optimizations in two days using a single GPU. The media dubbed it "Karpathy's Loop."

A new project called [auto-arch-tournament](https://github.com/FeSens/auto-arch-tournament/blob/main/docs/auto-arch-tournament-blog-post.md) asks what happens when you aim the same methodology at hardware. Not gradient descent, not hyperparameters — actual RISC-V microarchitecture, implemented in SystemVerilog.

The setup: a 5-stage in-order RV32IM pipeline, the kind of textbook core you'd write in a graduate architecture class. No caches, no branch predictor, no multi-issue. Locked-in baseline: 301 CoreMark iterations per second at 135 MHz. The agent's job is to make it faster.

The loop structure mirrors autoresearch. An orchestrator runs three parallel slots each round. A proposal agent generates microarchitectural hypotheses in YAML — things like "add a backward-branch-taken predictor to the instruction fetch stage" or "move DIV/REM off the single-cycle ALU into a dedicated multi-cycle unit." An implementation agent edits RTL files in isolated git worktrees. Then the critical gate: formal verification, cosimulation against a reference model, FPGA synthesis with place-and-route, and CoreMark validation. A change that passes correctness and shows positive CoreMark delta gets merged. Everything else gets discarded.

73 hypotheses. 10 winners. Just under 10 hours of wall-clock time.

The accepted improvements include a backward-branch-taken predictor and direct-jump prediction in the fetch stage, the DIV/REM restructuring (which paradoxically reduced LUT count by half — the agent discovered that decoupling the slow path from the hot ALU path helped synthesis optimize both), a one-deep store retirement slot, a banked I-fetch replay predictor, and a segmented RVFI performance counter. Final numbers: 577 iter/s, 199 MHz max frequency, 40% fewer LUTs than baseline. Against VexRiscv, a well-optimized open-source RISC-V core, the champion design runs 13% faster per MHz and 56% faster total.

A 14% acceptance rate on 73 proposals is not bad for experimental hardware changes. Architecture search is a hard problem — the space is large, most changes interact in non-obvious ways, and incorrect implementations are easy to generate. The agent did something useful.

But the author's most important observation is about something other than the agent: "the verifier is not commodity." The loop itself — propose, implement, measure — is straightforward to build. What makes the whole system work is the correctness gate. Formal verification against the ISA spec, cosimulation to catch behavioral regressions, FPGA synthesis to get real timing data. Without that pipeline, the agent optimizes whatever metric you give it while potentially producing hardware that fails in the field. With it, you get a tight feedback loop where "better" actually means correct and faster.

This is a pattern that comes up in every domain where autonomous research loops are applied. ML training loops work because loss is a reliable signal. Competitive programming loops work because test cases are definitive. Code generation loops work because test suites exist. The bottleneck is always the evaluator. Here, the evaluator is a full hardware verification stack — formal tools, simulator, FPGA — and that infrastructure took real effort to build. The agent loop is the easy part.

What the project demonstrates is that the methodology generalizes. RISC-V RTL is a long way from PyTorch training runs, but the same structure holds: propose a change, get a binary correct/incorrect signal plus a scalar performance metric, repeat. Whether that generalization extends to more complex designs — out-of-order cores, memory hierarchies, cache coherence — is an open question. At minimum, auto-arch-tournament provides a concrete existence proof that the loop works outside machine learning, and a blueprint for what the verification infrastructure needs to look like.
