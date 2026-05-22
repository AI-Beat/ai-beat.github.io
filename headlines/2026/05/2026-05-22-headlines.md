# AI Headlines — 2026-05-22

- [CODA: Rewriting Transformer Blocks as GEMM-Epilogue Programs](https://arxiv.org/abs/2605.19269) — Tri Dao and colleagues reparameterize the non-attention transformer ops (norms, activations, residuals) so they execute inside the matmul epilogue while output tiles are still on-chip, eliminating the DRAM round-trips that make these ops memory-bound; code released for H100 via CUTLASS CuTeDSL. *(May 19–20, 2026)*

- [KVBoost: chunk-level KV cache reuse for HuggingFace, 5–48× faster TTFT](https://pythongiant.github.io/KVBoost/) — Drop-in wrapper for HuggingFace Transformers that hashes prompt chunks and skips re-encoding previously seen prefixes; on a ShareGPT replay it holds TTFT flat at ~20ms across multi-turn sessions versus a baseline that climbs to 122ms by turn 8. *(May 20, 2026)*

- [Indexing a year of video locally with Gemma 4-31B on a 2021 MacBook](https://blog.simbastack.com/indexed-a-year-of-video-locally/) — Practical writeup of building a local video-indexing pipeline with WhisperX, ArcFace, and Gemma 4 running at 50GB swap; key finding: structured schemas (predefined option lists) eliminate certain confabulation patterns, and the local model matches Sonnet on most clips. *(May 21, 2026)*

- [Multi-Stream LLMs: Unblocking Language Models with Parallel Streams](https://arxiv.org/abs/2605.12460) — Architecture proposal to let LLMs maintain separate computation streams for inputs, thinking, and outputs, enabling genuine parallelism rather than sequential message exchange between agent components. *(May 12, 2026)*
