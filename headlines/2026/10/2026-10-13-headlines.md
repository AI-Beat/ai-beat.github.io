# AI Headlines — 2026-10-13

- [AI Models Can Provably Hide Arbitrary Capabilities](https://arxiv.org/abs/2610.13928) — Draguns et al. (NeurIPS 2026) embed encrypted neural networks inside model weights using cryptographic digital lockers; the hidden networks survive fine-tuning, weight noising, gradient-based trigger search, and latent adversarial training, placing a provable upper bound on what capability evaluations can guarantee. *(October 9, 2026)*

- [Codex CLI installs a persistent, self-updating background daemon by default](https://github.com/openai/codex/issues/48195) — Since version 0.157.0, `codex` auto-launches a 374 MB background process that keeps running after all windows close, self-updates hourly from a copy of itself in `~/.codex/packages/`, and continues even after `daemon stop`; community backlash calls for opt-in behavior. *(October 13, 2026)*

- [Janas: Run LLMs larger than RAM by streaming experts from SSD](https://github.com/prabanta-dev/janas) — C library with no dependencies streams Qwen3/Gemma4 MoE experts on demand from NVMe SSD; benchmarks show 37 tok/s for Qwen3-Next-80B-A3B (48.4 GB) on a 32 GB laptop, 13× faster than llama.cpp on the same hardware. *(October 13, 2026)*

- [A look at OpenAI's math that's claimed to compute faster Fourier transforms](https://www.argmin.net/p/faster-fourier-transforms) — Ben Recht digs into the OpenAI claim: the hidden constant is ~10^100 at N=10^12, the algorithm requires exact complex arithmetic at arbitrary precision, and the result is a theoretical exploit of loose computational modeling rather than a practical speedup. *(October 13, 2026)*
