# AI Headlines — 2026-06-03

- [MAI-Thinking-1: Microsoft's first in-house reasoning model, trained without OpenAI data](https://www.techtimes.com/articles/317631/20260602/microsoft-build-2026-mai-thinking-1-first-house-reasoning-model-trained-without-openai-data.htm) — Announced at Build 2026; ~1T total parameters (35B active via MoE), 256K context window, 97% AIME 2025; built entirely on commercially licensed data with no OpenAI distillation; available in private preview via Microsoft Foundry. *(June 2, 2026)*

- [MAI-Code-1-Flash: Microsoft's Copilot-native coding model](https://microsoft.ai/news/introducingmai-code-1-flash/) — Trained directly against GitHub Copilot production harnesses rather than generic benchmarks; 51.2% SWE-Bench Pro (vs. 35.2% for Claude Haiku 4.5); adaptive thinking adjusts reasoning budget per task; rolling out to all Copilot tiers in VS Code. *(June 2, 2026)*

- [Bringing Up DeepSeek-V4-Flash on AMD MI300X](https://fergusfinn.com/blog/deepseek-v4-flash-mi300x/) — Engineering account of getting DeepSeek's flash model running on MI300X: AMD's fnuz FP8 format causes off-by-factor-of-two errors vs. OCP standard, AITER kernels lack gfx942 coverage, HIP graphs conflict with sparse attention's dynamic metadata; result after fixes is ~2,700 output tokens/s per GPU. *(June 3, 2026)*

- [nbd-vram: Use your Nvidia GPU's VRAM as swap space on Linux](https://github.com/c0dejedi/nbd-vram) — A daemon allocates GPU memory via CUDA driver API and exposes it as a block device through the NBD protocol over Unix sockets; achieves ~1.3 GB/s sequential throughput on RTX 3070 Laptop (better than NVMe), effectively tripling available memory for LLM inference on VRAM-constrained hardware. *(June 3, 2026)*
