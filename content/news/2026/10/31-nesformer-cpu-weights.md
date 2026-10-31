---
title: "The CPU That Never Trained"
date: 2026-10-31T06:09:00+00:00
draft: false
slug: nesformer-cpu-weights
categories: [research]
tags: [research, architectures, inference, hardware]
params:
  author: AI Beat Desk
  summary: >-
    NESformer compiles a full 6502 CPU into the weights of a DeltaNet-based
    model, letting it run NES games without any gradient descent. The trick is
    that DeltaNet's recurrent state can be engineered row-by-row to act as
    addressable memory — turning the SSM's own hidden state into a 2KB RAM
    that the CPU can read and write deterministically.
---

There is a [new project on GitHub](https://github.com/dbreunig/nesformer) in which Super Mario Bros. runs inside a neural network. Not *controlled* by one, not *played* by one — the actual CPU emulation, the 6502 instruction fetch-decode-execute loop, happens inside a transformer's forward pass. The project is called nesformer, and it's a direct successor to Qwenputer, which did the same trick for CHIP-8 and a simple RISC-V core.

The first question is always "but isn't the model just *running* the emulator in Python?" No. The 6502 CPU, its 2 KB of work RAM, and the cartridge's program ROM are encoded in the model's weights. During inference, the model predicts the CPU's execution log one token at a time. Each forward pass is a clock cycle. There is no emulator hiding in the host code; the host program only handles what the NES hardware outside the CPU does — drawing pixels, reading the controller, firing the vertical blank interrupt.

The mechanism hinges on DeltaNet, a linear recurrent attention variant that replaces the standard softmax attention head with a low-rank state-space update. Each DeltaNet layer maintains a hidden state matrix, and the key observation is that this state matrix can be directly engineered: set each row to represent a specific memory cell, and the layer's update rule will read and write those cells deterministically given the right keys and queries. The compiler that builds nesformer's weights doesn't train anything — it solves for the weight matrices analytically, wiring the DeltaNet state rows to correspond to the 6502's 2 KB address space plus its internal registers.

The resulting checkpoint loads with Hugging Face `transformers` as a standard `Qwen3_5ForCausalLM`, but it's not the Qwen3.5-4B model. It borrows that architecture's DeltaNet head shapes and MLP width, but the model has a hidden size of 768 and roughly 466 million parameters total — of which only about 9.3 million are nonzero. The weights file is approximately 1 GB. To put that in perspective: the NES has 2 KB of RAM. The model needs about a hundred thousand times more storage to simulate it, but it *does* simulate it, correctly, one 6502 instruction at a time.

Why does any of this matter? The direct practical answer is that it doesn't — this is not a better way to play old games. But the indirect answer is interesting.

The SSM literature has spent the last few years arguing that state space models are more amenable to hardware-efficient inference than softmax attention, and that their recurrent form gives them theoretical advantages for long-context tasks. Nesformer is, in a sense, a stress test of that recurrent state: if you can deterministically route memory reads and writes through the DeltaNet hidden state, the architecture is Turing-complete in a very concrete operational sense. You can compile any sequential digital circuit into it.

That's related to, but distinct from, theoretical universality results about neural networks. Those results typically say *there exist* weights that compute a given function. Nesformer shows *how* to find those weights for a specific, well-specified computation — and the "how" is a compiler pass, not a training run. The technique generalizes wherever the target computation is fully specified in advance and the architecture's state update is invertible enough to encode memory addressing.

There are real limits. Nesformer only supports NROM cartridges (iNES mapper 0), which is the simplest NES memory configuration with no bank switching. The CPU runs at the pace of the model's token generation, not at the NES's actual 1.79 MHz clock, so the video and timing behavior have to be approximated by the host. And the 466M-parameter count, though almost entirely zeros, makes the checkpoint large for what is fundamentally less than 3 KB of CPU state.

Still, the compiler-driven approach to weight setting is worth watching. The gradient-free direction in neural network research has mostly focused on prompt engineering, in-context learning, or retrieval — techniques that don't change the weights at all. Nesformer is in a smaller and older tradition of understanding what weight spaces actually contain, and what circuits you can express there on purpose rather than by accident.

The code and the compressed weights are on [GitHub](https://github.com/dbreunig/nesformer).
