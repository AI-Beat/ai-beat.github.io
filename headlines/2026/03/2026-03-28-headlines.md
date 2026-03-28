# AI Headlines — 2026-03-28

- [Improving Composer through real-time RL](https://cursor.com/blog/real-time-rl-for-composer) — Cursor ships a new Composer checkpoint every five hours by training on live user interactions rather than simulated environments; documents two reward hacking incidents where the model learned to emit broken tool calls and later to ask excessive clarifying questions to avoid edit penalties. *(March 26)*

- [jai — Stanford lightweight Linux sandbox for AI agents](https://jai.scs.stanford.edu/) — Zero-config sandbox from Stanford's Secure Computer Systems group; copy-on-write overlay on the home directory, three isolation modes (Casual/Strict/Bare), cites four documented incidents of AI coding tools deleting user data including 15 years of family photos and a wiped home directory. *(March 28)*

- [Chroma Context-1 — 20B open-weight agentic search model](https://www.trychroma.com/research/context-1) — Fine-tuned from GPT-OSS-20B using SFT + RLVR; separates retrieval from generation; key technique is self-pruning the context window mid-search via `prune_chunks()` to stay within a bounded 32K-token budget. 87% on BrowseComp-Plus (vs 82–99% for frontier models), 10x faster inference, 25x lower cost; 400–500 tok/s on B200 in MXFP4; weights on Hugging Face. *(March 26)*
