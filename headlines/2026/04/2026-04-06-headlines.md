# AI Headlines — 2026-04-06

- [GrandCode: Achieving Grandmaster Level in Competitive Programming via Agentic Reinforcement Learning](https://arxiv.org/abs/2604.02721) — DeepReinforce Team reports an AI system that placed 1st in three consecutive live Codeforces rounds in March 2026, beating all human participants including legendary grandmasters; key contributions are a multi-agent orchestration system and "Agentic GRPO," a GRPO variant designed for multi-stage rollouts with delayed rewards. *(April 3, 2026)*

- [FoE: Forest of Errors Makes the First Solution the Best in Large Reasoning Models](https://arxiv.org/abs/2604.02967) — Identifies a counterintuitive pattern in large reasoning models: the first solution in a chain-of-thought is consistently the best, and subsequent alternatives tend to degrade performance due to error propagation; proposes RED (Refine and Discard) which achieves up to 19% performance gains while reducing token usage by 37–70%. *(April 3, 2026)*

- [GuppyLM: Tiny 8.7M-parameter LLM as a teaching tool](https://github.com/arman-bd/guppylm) — A deliberately minimal vanilla transformer (6 layers, 384-dim, 4K BPE vocab, 128-token context) trained on 60K synthetic conversations; trains in ~5 minutes on a free Colab T4 GPU; no RoPE, no GQA, no SwiGLU — purely a self-contained demonstration of the full LLM training pipeline. *(March 29, 2026; HN front page April 6, 2026)*

- [Gemma Gem: In-browser autonomous agent running Gemma 4 on-device via WebGPU](https://github.com/kessler/gemma-gem) — Chrome extension that runs Gemma 4 E2B/E4B entirely on-device (no API keys); capable of reading page content, taking screenshots, clicking, and filling forms as a browser agent; built on HuggingFace Transformers.js with q4f16 quantization and 128K context. *(April 5, 2026)*

- [Hallucination as Cue: RL post-training on corrupted visual inputs still improves multimodal reasoning](https://arxiv.org/abs/2604.03179) — CVPR 2026 paper from Gengwei Zhang et al.: RL training of multimodal models on modality-corrupted inputs (removed or replaced images) still substantially improves reasoning, challenging assumptions about how visual grounding develops during RL post-training. *(April 3, 2026)*

- [Gradient Boosting within a Single Attention Layer](https://arxiv.org/abs/2604.03190) — Proposes a second attention pass with separate learned projections that corrects residual errors from the first, drawing a formal connection to Friedman's gradient boosting; achieves perplexity 67.9 vs. 72.2 for standard attention on WikiText-103. *(April 3, 2026)*
