# AI Headlines — 2026-04-03

- [Cursor 3: Unified workspace for building software with agents](https://cursor.com/blog/cursor-3) — Cursor 3 reframes the IDE as a multi-agent orchestration platform: all local and cloud agents (initiated from mobile, web, Slack, GitHub, Linear) appear in a unified sidebar; Composer 2 is Cursor's in-house frontier coding model; new Marketplace for plugins/MCPs/subagents; cloud↔local handoff lets you keep agents running offline. *(April 2, 2026)*

- [Google Gemma 4: Open-weight multimodal family built on Gemini 3](https://blog.google/technology/developers/google-gemma-4/) — Four models under Apache 2.0: edge-optimized E2B and E4B with 128K context; 26B sparse MoE (3.8B active, 256K context); 31B Dense; built on Gemini 3 architecture; supports text/audio/image inputs in 140 languages; native function calling; deployable via Ollama, HuggingFace, Kaggle, Docker; NVIDIA-optimized for local RTX hardware. *(April 2, 2026)*

- [Microsoft MAI-Transcribe-1, MAI-Voice-1, MAI-Image-2](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/introducing-mai-transcribe-1-mai-voice-1-and-mai-image-2-in-microsoft-foundry/4507787) — Microsoft's first internally built foundational models, launching through Azure AI Foundry: MAI-Transcribe-1 covers 25 languages at 50% lower GPU cost than competitors ($0.36/hr); MAI-Voice-1 generates 60s of audio in under 1s on a single GPU ($22/1M chars); MAI-Image-2 debuted at #3 on Arena.ai image leaderboard. *(April 2, 2026)*

- [Lemonade by AMD: Open-source local LLM server with GPU+NPU support](https://lemonade-server.ai) — AMD's open-source local AI platform; 2MB native C++ backend; auto-configures across Windows/Linux/macOS; runs text, image, and speech models simultaneously on consumer GPU and NPU; OpenAI API compatible; built on llama.cpp and ONNX Runtime; targets Ryzen AI and Radeon hardware. *(April 2–3, 2026)*

- [LocalAI v4.1.0: Distributed cluster support and fine-tuning](https://github.com/mudler/LocalAI/releases) — v4.1.0 adds distributed cluster smart routing and autoscaling, OIDC-based user management with per-user quotas, experimental TRL fine-tuning with GGUF auto-export, a visual pipeline editor, and standalone CLI agents. *(April 2, 2026)*

- [Qwen3.6-Plus: Alibaba's agent-focused model with 0.904 GPQA](https://qwen.ai/blog?id=qwen3.6) — Alibaba's latest proprietary model positioned toward real-world agentic use cases; strong GPQA benchmark performance (0.904); third Qwen release in less than a week. *(April 1, 2026)*

- [Claude Code prompt caching bug caused 25–50x cost overruns (fixed in v2.1.88)](https://www.emsi.me/tech/ai-ml/10509/2026-03-31/113a11) — A bug caused tool schema bytes to change mid-session, invalidating the KV cache on every turn; users burning through Max plan limits; each turn forced reprocessing of 200K+ cached tokens; initially misattributed to peak-hour throttling; fixed in v2.1.88. *(March 31, 2026)*

- [ROS-LLM: Natural-language robot control published in Nature Machine Intelligence](https://www.nature.com/articles/s42256-026-01186-z) — Framework from Huawei Noah's Ark Lab, TU Darmstadt, and ETH Zurich combining ROS with LLMs for natural-language robot control; supports both inline code and behavior tree execution; learns new skills via imitation; claims 10-minute integration time for new robots. *(April 2026)*

- [Anthropic acquires Coefficient Bio for ~$400M](https://llm-stats.com/ai-news) — Anthropic acquires biotech platform aimed at automating drug research planning using AI. *(April 3, 2026)*
