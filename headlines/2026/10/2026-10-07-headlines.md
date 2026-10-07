# AI Headlines — 2026-10-07

- [OpenTPU – An open-source AI accelerator, developed by AI](https://github.com/FeSens/openTPU) — FeSens built a complete FPGA-based AI inference accelerator (RTL, ISA, compiler, profiler) using AI agents, targeting the question "can AI agents build the chip that runs their own inference?"; runs on a Kintex-7 xc7k480t FPGA achieving 85.8 tok/s decode for LFM2.5-230M and 12 tok/s for Qwen3.5-2B; supports 18 model configs including Qwen3, Gemma 4, and Phi-4-mini. *(October 2026)*

- [EmbeddingGemma 2: An open, lightweight multimodal embedding model](https://blog.google/innovation-and-ai/technology/developers-tools/embeddinggemma-2/) — Google releases a 740M-parameter embedding model (270M text-only config) that maps text, code, images, video, and audio to a single shared latent space; built on Gemma 4 with 8K context, Matryoshka Representation Learning (768→128 dims), and MTEB code score of 78.68 (up 9.92 from predecessor); Apache 2.0. *(October 6, 2026)*

- [Strands Decider 2B: a small, open-source, decision model](https://strandsagents.com/blog/introducing-strands-decider/) — AWS Strands Agents team releases Decider 2B, a classification-only model derived from Qwen 3.5-2B (generation head replaced with a scoring mechanism) that returns typed answers in a single parallel pass at ~115ms median latency on an RTX 3090; ranked 3rd on JevBench among 2B models. *(October 1, 2026)*

- [OpenAI Decisions API in public beta](https://developers.openai.com/api/docs/guides/decisions) — New API that evaluates text/images to produce typed answers (predicate probability, choice selection, or score) "10x faster than the Responses API" using gpt-6-luna; charges only for input tokens ($0.10/M); designed for classification, routing, and quality inspection workflows. *(October 2026)*

- [Mistral Large 4](https://mistral.ai/news/mistral-large-4/) — Mistral releases a 1T-parameter natively multimodal MoE model (49B active params, "le Chonk") trained on 3,800 Grace Blackwell GPUs; strong coding (61.7% DeepSWE), cybersecurity, and visual grounding; API preview live, open weights and Apache 2.0 release by end of October. *(October 6, 2026)*

- [Sharing AI progress in mathematics](https://openai.com/index/sharing-ai-progress-in-mathematics/) — OpenAI update on its Astra model's progress in mathematics, continuing from the August 2026 announcement where it solved 10 decade-open problems; new results reportedly cover 100+ problems across most areas of mathematics. *(October 7, 2026)*
