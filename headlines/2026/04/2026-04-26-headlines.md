# AI Headlines — 2026-04-26

- [LamBench: Lambda Calculus Benchmark for AI](https://victortaelin.github.io/lambench/) — Victor Taelin (HVM/Interaction Calculus) releases 120-problem pure lambda calculus benchmark in the Lamb language; top models (GPT-5.3 Codex, Opus 4.6) score 90%, while GPT-5.1, Opus 4.5, and Sonnet 4.5 score 0/120, revealing a sharp generational cliff in symbolic reasoning ability. *(April 23, 2026)*

- [How Much Is One Recurrence Worth? Scaling Laws for Looped Language Models](https://arxiv.org/abs/2604.21106) — 116 pretraining experiments characterize a "recurrence-equivalence exponent" φ = 0.46: looping a block r times is equivalent in validation loss to r^0.46 unique blocks; a 410M looped model trained at 1B-model compute matches a 580M non-looped model, with looped architectures preferring wider, shallower configurations. *(April 24, 2026)*

- [Hyperloop Transformers: Parameter-Efficient Looped Architectures with Hyper-Connections](https://arxiv.org/abs/2604.21254) — Splits the transformer into begin/middle/end blocks where only the middle repeats, adding matrix-valued hyper-connections that condition on loop iteration; a 579M Hyperloop model outperforms a 1B standard transformer on both perplexity and nine downstream tasks, with the advantage persisting through INT4 quantization. *(April 24, 2026)*
