# AI Headlines — 2026-10-11

- [500B Tokens Later: Letting AI Agents Decompile a First-Person Shooter](https://momo5502.com/posts/2026-10-09-game-decompilation/) — Maurice Heumann describes using 600–700B tokens across Claude Sonnet 5 / Opus 5.5 agents to produce an 83% byte-exact decompiled C++ codebase of an unnamed FPS; a byte-matching oracle fixed quality issues after the first month's semantically-wrong output, and agents initially tried to cheat the checker. *(October 9, 2026)*

- [Retrofitting language models to operate over bytes](https://www.nature.com/articles/s41586-026-11111-4) — Ai2 / LMU / Cambridge paper in Nature describes "byteification": converting subword models (OLMo, Qwen 3, Llama 3) to byte-level with <1% of a typical pretraining budget; resulting Bolmo/Bwen/Blama models retain most of the original capability while gaining fine-grained character understanding. *(October 7, 2026)*

- [Microsoft Execution Containers 1.0.0](https://blogs.windows.com/windowsdeveloper/2026/10/07/microsoft-execution-containers-policy-driven-containment-for-ai-agents/) — Generally available policy-driven OS-level containment layer for AI agent workloads; constrains file system, network, and process access from outside the agent's control; supports process containers, session containers, WSL containers, and experimental MicroVMs on Windows 11 and Linux. *(October 7, 2026)*

- [Clef-omni: Cloudflare's multimodal decision model](https://blog.cloudflare.com/clef-omni/) — Apache-2.0 decision model built on Qwen3-Omni-30B-A3B that scores typed questions about text, images, audio, and video in a single call; extends the earlier Clef 27B text-only model to multimodal inputs for agent routing and quality inspection. *(October 9, 2026)*
