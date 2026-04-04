# AI Headlines — 2026-04-04

- [Claude Code Found a Linux Vulnerability Hidden for 23 Years](https://mtlynch.io/claude-code-found-linux-vulnerability/) — Nicholas Carlini (Anthropic) pointed Claude Opus 4.6 at Linux kernel source files one at a time; it surfaced five confirmed CVEs including an NFSv4.0 heap overflow that had been in the kernel since 2003, plus a working remote root exploit in FreeBSD. *(April 3, 2026)*

- [Anthropic ends Claude Code subscription coverage for third-party harnesses like OpenClaw](https://venturebeat.com/technology/anthropic-cuts-off-the-ability-to-use-claude-subscriptions-with-openclaw-and) — Effective April 4, Claude Code Max/Pro subscribers using OpenClaw or other external agents must pay Extra Usage on top of their subscription; Boris Cherny (Head of Claude Code) cited unsustainable compute demand from third-party usage patterns. *(April 4, 2026)*

- [Embarrassingly Simple Self-Distillation Improves Code Generation](https://arxiv.org/abs/2604.01193) — Sample the model at high temperature, filter by execution success, SFT on the results — no teacher, no RL, no reward model; lifts Qwen3-30B from 42.4% to 55.3% pass@1 on LiveCodeBench v6 and generalizes across Qwen/Llama at 4B–30B scale. *(April 1, 2026)*

- [Show HN: TurboQuant-WASM — Google's KV cache compression running in the browser](https://github.com/teamchong/turboquant-wasm) — Zig+WASM port of Google's TurboQuant (ICLR 2026): 3 bits/dim vector quantization with fast dot product using relaxed SIMD; ~6× compression on float arrays; includes a live demo with vector search and image similarity. *(April 4, 2026)*
