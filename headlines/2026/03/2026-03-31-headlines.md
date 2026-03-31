# AI Headlines — 2026-03-31

- [Ollama is now powered by MLX on Apple Silicon (preview)](https://ollama.com/blog/mlx) — Ollama switches from direct Metal to Apple's MLX framework on M5 chips; Qwen3.5-35B-A3B decode goes from 58 to 112 tokens/s, prefill from 1,154 to 1,810 tokens/s; adds NVFP4 quantization support and smarter KV cache management; requires >32GB unified memory. *(March 30)*

- [Mr. Chatterbox: A language model trained entirely on Victorian public domain texts](https://simonwillison.net/2026/Mar/30/mr-chatterbox/) — 340M-param model trained on 28,000+ British Library books (1837–1899) using Karpathy's nanochat; 2.93B training tokens, 2.05GB on disk; Chinchilla-undercooked and not conversationally useful, but a principled proof-of-concept for copyright-clean LLM training. *(March 30)*

- [Microsoft Harrier-OSS-v1: Multilingual embedding models with 32K context, SOTA on MTEB v2](https://huggingface.co/microsoft/harrier-oss-v1-0.6b) — Decoder-only embedding family (270M / 0.6B / 27B) with a 32,768-token context window — vs. typical 512–1,024 for most embedding models; MIT licensed; 94 languages; 27B variant scores 74.3 on Multilingual MTEB v2 (SOTA); smaller models use knowledge distillation from the 27B. *(March 30)*

- [Qwen3.5-Omni: Alibaba open-weights omnimodal model with 256K context](https://decrypt.co/362742/alibaba-qwen-omni-major-upgrade-review) — Processes text, image, audio, and video input with real-time speech output; 256K token context (10+ hours of audio); speech recognition in 113 languages and generation in 36; Plus/Flash/Light variants; new ARIA technique for speech accuracy; outperforms ElevenLabs and GPT-Audio on multilingual voice stability. *(March 30)*
