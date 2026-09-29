# AI Headlines — 2026-09-29

- [Jeff: Jev-compatible 0.8B–2B decision models, trained at home](https://github.com/firelex/jeff) — Firelex fine-tuned Qwen3.5 and Gemma 4 for zero-shot classification using synthetic training data, full-weight fine-tuning, and temperature calibration; 2B model hits 83.1% on five public benchmarks (Jev scores 83.0%), trained in 2–3.5 hours on a single RTX PRO 6000. *(September 28, 2026)*

- [ESP32S3-LLM-Cluster: a 0.5B BitNet model sliced across 7 microcontrollers](https://github.com/Low-Zi-Hong/ESP32s3-LLM-Cluster) — Daisy-chain SPI topology; master handles tokenization/embedding, six compute nodes process 4 transformer layers each using 1.58-bit ternary weights in flash; a concrete proof that BitNet extends to sub-$30 microcontroller hardware. *(September 2026)*

- [FragToken: training-time attack that inflates LLM inference costs 2–2.5× without visible output growth](https://arxiv.org/abs/2609.31552v1) — Self-distillation + filtering pipeline trains a model to emit noncanonical multi-token sequences where single tokens would suffice, doubling decoding steps while preserving output quality; authors call it a covert LLM supply-chain threat. *(September 25, 2026)*
