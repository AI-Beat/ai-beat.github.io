# AI Headlines — 2026-06-05

- [Do Transformers Need Three Projections? Systematic Study of QKV Variants](https://arxiv.org/abs/2606.04032) — ICML 2026 paper showing that sharing key and value projections (Q-K=V) halves the KV cache with only 3.1% perplexity degradation; combined with MQA, achieves 96.9% cache reduction; keys and values can occupy similar representational spaces while sharing the query projection breaks attention directionality. *(June 1, 2026)*

- [KVarN: Native vLLM backend for KV-cache quantization by Huawei](https://github.com/huawei-csl/KVarN) — Calibration-free asymmetric quantization (4-bit keys, 2-bit values) using Hadamard rotation and iterative variance normalization; 3-5x more KV-cache capacity with throughput above FP16, one-flag integration into vLLM, state-of-the-art on MATH500 and AIME24 at 2-bit precision. *(June 2, 2026)*

- [Magenta RealTime 2: Open and Local Live Music Models](https://magenta.withgoogle.com/magenta-realtime-2) — Google's open-weights live music generation model runs locally on Apple Silicon MacBooks; a decoder-only causal architecture cuts control latency from ~3s to ~200ms; the 2.4B base model requires M3 Pro or M2 Max, 230M small model runs on any Apple Silicon. *(June 4, 2026)*

- [Anthropic's open-source framework for AI-powered vulnerability discovery](https://github.com/anthropics/defending-code-reference-harness) — Reference implementation of an autonomous vulnerability pipeline using Claude: recon → find → verify → dedupe → patch, targeting C/C++ memory bugs via ASAN inside gVisor sandboxes; comes with Claude Code slash commands for interactive use. *(June 5, 2026)*
