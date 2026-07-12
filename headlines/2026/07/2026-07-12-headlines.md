# AI Headlines — 2026-07-12

- [Mesh LLM: distributed AI computing on iroh](https://www.iroh.computer/blog/mesh-llm) — Pools GPU and memory across consumer machines via iroh's P2P networking layer (QUIC, no central server); requests route locally, to a peer with the model already loaded, or split by layer range across multiple nodes via the "Skippy" engine; works well on LAN but latency kills it over the internet. *(July 11, 2026)*

- [An agent in 100 lines of Lisp](https://thebeach.dev/posts/lisp-agent/) — Instead of a predefined toolkit, the agent gets one tool — eval — and builds everything else by writing Common Lisp into the runtime; capabilities persist across sessions by re-evaluating function definitions stored in the JSON transcript; the model spontaneously created a working web search client when given API credentials. *(July 7, 2026)*

- [Mindwalk – replay coding-agent sessions on a 3D map of your codebase](https://github.com/cosmtrek/mindwalk) — Visualizes Claude Code and Codex session logs as illuminated paths across a tree or terrain view of a repository, with color-coded touch states (unvisited/viewed/read/edited) and timeline scrubbing; fully local, no session data leaves your machine. *(July 11, 2026)*
