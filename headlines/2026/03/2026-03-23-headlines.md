# AI Headlines — 2026-03-23

- [If DSPy is So Great, Why Isn't Anyone Using It?](https://skylarbpayne.com/posts/dspy-engineering-patterns/) — DSPy has 4.7M monthly downloads vs LangChain's 222M. Skylar Payne argues teams inevitably reinvent DSPy's patterns (typed I/O, composable modules, eval infra) through seven painful stages, but do it worse. *(March 21)*

- [Walmart: ChatGPT in-chat checkout converted at 1/3 the rate of click-outs](https://searchengineland.com/walmart-chatgpt-checkout-converted-worse-472071) — Walmart tested OpenAI's Instant Checkout across ~200K products; in-chat purchases converted 3x worse than sending users to Walmart.com. Now pivoting to Sparky-in-ChatGPT for merchant-controlled checkout. *(March 19)*

- [iPhone 17 Pro runs 397B Flash-MoE at 0.6 tokens/sec](https://wccftech.com/iphone-17-pro-successfully-runs-400b-llm-locally/) — @anemll demo showing the Flash-MoE SSD-streaming technique on an iPhone 17 Pro (12GB RAM). Technically works; practically a proof-of-concept at current speed. *(March 23)*

- [Flash-MoE — 397B parameter Qwen3.5 MoE running on a MacBook Pro at 4.4 tokens/sec](https://github.com/danveloper/flash-moe) — Pure C/Metal inference engine, no Python, no frameworks. Streams the 209GB model from SSD, loading only 4 of 512 experts per token. First open-source implementation of Apple's "LLM in a Flash" technique at usable speed. *(March 18)*

- [NVIDIA Nemotron-Cascade 2 — 30B MoE with 3B active params, gold medal on IMO 2025](https://research.nvidia.com/labs/nemotron/nemotron-cascade-2/) — Open-weight, fits on a single RTX 4090 (24GB VRAM at Q4). Achieves IMO 2025 gold (35/42) and IOI 2025 gold — only the second open model ever to do so, after a 671B DeepSeek. Trained via "Cascade RL" sequential domain training; full weights, data, and paper open. *(March 19–20)*

- [Sarvam 30B & 105B — India's sovereign open-weight reasoning models](https://www.sarvam.ai/blogs/sarvam-30b-105b) — Trained entirely on IndiaAI Mission compute, covering 22 Indian languages. The 105B uses DeepSeek-style MLA and outperforms DeepSeek R1, Gemini 2.5 Flash, and o4-mini on Tau 2 Bench. Apache 2.0. Developer adoption currently hindered by missing GGUF formats and limited vLLM support. *(March 2026)*

- [Helios — 14B open video model generating 60-second clips at 19.5 FPS on one H100](https://pku-yuangroup.github.io/Helios-Page/) — Joint release from Peking University, ByteDance, and Canva. Only 3 denoising steps via adversarial hierarchical distillation; runs on ~6GB VRAM with Group Offloading. Apache 2.0. *(March 4, trending week of March 18)*
