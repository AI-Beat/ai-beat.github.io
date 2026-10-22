---
title: "A Terminal With No Shell"
date: 2026-10-22T06:15:00+00:00
draft: false
slug: worldtui-terminal-world-model
categories: [research]
tags: [world-models, fine-tuning, webgpu, inference]
params:
  author: AI Beat Desk
  summary: >-
    WorldTUI is a 360M-parameter model fine-tuned to predict terminal screen
    changes from keystrokes, with no shell running behind it. It scores 80/106
    on ShellBench and runs in-browser at int4 via WebGPU. As a demo of narrow
    world modeling it's impressive; as a reliable terminal it isn't — but that
    gap is the interesting part.
---

[WorldTUI](https://simedw.com/2026/10/21/world-model-terminal/) is a 360M-parameter model that pretends to be a terminal. There is no shell. No filesystem. When you type a command, the model predicts how the screen should change. The author's framing is honest: "a pretty weak world model." That undersells the interesting parts.

The model is fine-tuned from Qwen2.5-Coder-0.5B on roughly 336k synthetic sessions — about two billion tokens — recorded from real programs running in Docker containers with randomized users, directories, and terminal sizes. The vocabulary is a tight 333 tokens representing row rewrites, cursor movements, and a `COPY` operation for scroll behavior. Training took 75 GPU-hours on an RTX 4090.

The core representation is a character grid. The model takes a screen state plus a keypress and outputs a diff of the screen. Rows are decoded in parallel, with speculative decoding patching inconsistencies between them. At int4 quantization the model is 208 MB and runs in the browser through a custom WebGPU implementation. On an M3 MacBook the p95 latency for a large frame is under 1 second.

**Where it works.** On ShellBench it scores 80/106; prompted baselines (an LLM told to simulate a terminal) score 52 and 60. The really striking result is file memory: 9/13 correct versus 2–3 for the baselines. The model can track what you wrote to a file and read it back plausibly — not because it has a filesystem, but because the training data taught it what `cat` usually does after `echo`. It handles vim, tmux (basic split operations), the snake game, environment variables, and a Python REPL at a level that fooled the benchmark consistently.

**Where it doesn't.** Complex Python answers are often hallucinated. File appends don't always survive a `cat`. Deep nested directories cause drift. The author tried distilling to a smaller architecture for the memory component and gave it up when distillation reliably destroyed retention.

What's being tested here isn't whether the model is a useful shell replacement — it obviously isn't. What's being tested is whether a small model can learn a domain-specific world model from synthetic data and self-correction feedback. The DAgger-style pass (the model is shown its own mistakes and trained to fix them) raised [PresetBench](https://simedw.com/2026/10/21/world-model-terminal/) from 24/47 to 38/47 at int4. That's a meaningful gain from a relatively cheap step.

The terminal is a useful domain for this kind of experiment precisely because it is constrained. The state space is bounded — a character grid with a cursor — and the transitions are mostly deterministic. A model that learns it well has actually learned something about the structure of unix tools, which is why it retains file contents better than a prompted baseline that has no persistent state to track.

There are cleaner ways to solve the problem the benchmark tests (a real filesystem, for instance), but that's not the point. The question is how much latent structure a small fine-tuned model can learn from execution traces, and the answer here is: enough to be interesting.

Weights haven't been posted yet. The author says they will be.
