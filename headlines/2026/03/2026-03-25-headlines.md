# AI Headlines — 2026-03-25

- [Arm AGI CPU: 136-core Neoverse V3, TSMC 3nm, Meta as first customer](https://newsroom.arm.com/blog/introducing-arm-agi-cpu) — Arm's first production silicon in its 35-year history; 300W part with 136 cores at 3.7 GHz, DDR5-8800, >800 GB/s bandwidth, CXL 3.0/PCIe Gen 6; designed for agentic AI CPU-side orchestration; OpenAI, Cloudflare, Cerebras among launch partners. *(March 24)*

- [TurboQuant: Google Research blog on KV cache compression](https://research.google/blog/turboquant-redefining-ai-efficiency-with-extreme-compression/) — ICLR 2026 paper; compresses KV cache to ~3 bits with zero accuracy loss and no fine-tuning; 6x memory reduction, up to 8x speedup on H100 at 4-bit; combines PolarQuant (polar coordinate quantization) + QJL (1-bit Johnson-Lindenstrauss error correction); proved information-theoretically near-optimal. *(March 25)*

- [Show HN: Hypura — storage-tier-aware LLM inference scheduler for Apple Silicon](https://github.com/t8/hypura) — Runs models larger than Mac RAM by tiling across GPU, RAM, and NVMe; MoE-aware (loads only active experts per token, 99.5% temporal cache hit rate); Mixtral 8x7B at 2.2 tok/s on M1 Max where llama.cpp OOMs; Ollama-compatible API. *(March 25)*
