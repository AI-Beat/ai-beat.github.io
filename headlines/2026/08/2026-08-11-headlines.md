# AI Headlines — 2026-08-11

- [LFM2.5-2.6B: On-Device Agent Model from Liquid AI](https://huggingface.co/blog/LiquidAI/lfm2-5-2-6b) — Liquid AI releases a 2.6B-parameter model that runs a full tool-calling agent in under 2.5 GB of RAM; 220 tok/s on Apple M5 Max CPU and 30 tok/s on a mid-range phone, trained with a four-stage post-training pipeline ending in agentic RL; beats models up to 4x its size on ToolSandbox and IFStruct. *(August 4, 2026)*

- [Needle2: 14MB Agentic LLM for Phones, Wearables, and Microcontrollers](https://cactuscompute.com/needle) — Cactus Compute ships a 45M-parameter Simple Attention Network (feed-forward layers removed entirely) quantized to CQ2-bit from the start of training; the resulting 14MB binary runs a full inference session in 28MB of RAM and handles tool calling, device control, and structured extraction on ESP32-S3 microcontrollers. *(August 11, 2026)*

- [antirez/h3.c: Native C+Metal Inference Engine for MiniMax H3](https://github.com/antirez/h3.c) — Salvatore Sanfilippo (creator of Redis) releases a native Metal inference engine for MiniMax H3 (33B open-weight video+audio model) targeting M3/M5 Macs; ~75-second renders at 40GB peak memory, MIT-licensed, with ongoing H3-specific Metal kernel optimization. *(August 11, 2026)*
