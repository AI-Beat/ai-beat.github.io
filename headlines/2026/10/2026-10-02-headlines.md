# AI Headlines — 2026-10-02

- [Context Language Models](https://arxiv.org/abs/2609.37725) — Meta AI / Allen AI paper proposes giving models read-write access to their own context as a plain file; CLMs use Bash tools to edit, compress, and reorganize their working notes mid-task, cutting FLOPs by up to 59% on long-horizon benchmarks while improving accuracy. *(September 29, 2026)*

- [Clef: Cloudflare's open-weight decision models and RL fine-tuning platform](https://blog.cloudflare.com/clef-decision-models/) — Two open-weight classification models (Clef 27B and Clef-flash 9B) for agent routing and classification, hosted on Workers AI and released Apache 2.0; ships with a managed RL fine-tuning service for domain-specific adaptation. *(October 1, 2026)*

- [Pi 1.0](https://earendil.com/posts/pi-1-0/) — Earendil's minimal agent harness reaches stable 1.0 with Codemode (MCP and non-LLM model support), deferred tool loading, cache warming, and mid-conversation system message updates; companion Pi Durable package adds persistence, checkpointing, and background compaction for long-running agents. *(October 1, 2026)*

- [K-Dense BYOK](https://github.com/K-Dense-AI/k-dense-byok) — Open-source AI research assistant for scientists that runs locally with a bring-your-own-keys model; keeps a hash-chained Living Lab Notebook the agent cannot overwrite, includes 170+ scientific skills and 326 workflow templates; MIT-licensed. *(September 30, 2026)*

- [Claude Code Mods](https://x.com/ClaudeDevs/status/2105721434807083061) — Claude Code 2.1.287 introduces in-process TypeScript hooks that can intercept, rewrite, and answer agent events (tool calls, prompts, permission requests, UI renders); mods load once per session, keep state, and expose a sandbox API covering UI, routing, secret redaction, and custom slash commands. *(October 1, 2026)*

- [You should know — Claude Code's built-in observer mod](https://x.com/ClaudeDevs/status/2106118517447876618) — Claude Code 2.1.288 ships a built-in mod that runs a side agent alongside your main session, watching for things you or Claude might miss (secret exposure, destructive ops, outward-facing actions); enable with `/plugin enable cc-plugin-you-should-know@builtin`; requires telemetry. *(October 2, 2026)*
