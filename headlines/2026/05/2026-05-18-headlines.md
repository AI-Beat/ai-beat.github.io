# AI Headlines — 2026-05-18

- [Semble: Code Search for Agents Using ~98% Fewer Tokens Than grep](https://github.com/MinishLab/semble) — Combines Model2Vec static embeddings with BM25, fused via Reciprocal Rank Fusion and reranked using code-aware signals; indexes a repo in 263ms, answers queries in 1.5ms on CPU; ships as an MCP server drop-in for Claude Code, Cursor, and Codex. *(May 12, 2026)*

- [Argus: Evidence Assembly for Scalable Deep Research Agents](https://arxiv.org/abs/2605.16217) — Separates a ReAct-style Searcher from an RL-trained Navigator that maintains an evidence graph, identifies gaps, and dispatches purposeful parallel searches; achieves 86.2 on BrowseComp with 64 parallel Searchers while keeping Navigator context under 21.5K tokens. *(May 15, 2026)*

- [Reasoning Models Don't Just Think Longer, They Move Differently](https://arxiv.org/abs/2605.15454) — After length correction, reasoning-trained models show distinct hidden-state trajectory geometry compared to standard models, with the effect strongest in code tasks; correlates with strategy shifts and uncertainty monitoring, establishing length correction as necessary for comparing model internals. *(May 14, 2026)*

- [Zerostack: A Unix-Inspired Coding Agent Written in Pure Rust](https://github.com/gi-dellav/zerostack) — Minimal coding agent (~7k LoC, 8.9MB binary, ~8–12MB idle RAM); v1.1.0 released today with multi-provider LLM support, configurable permission modes, MCP integration, ACP editor protocol, and git worktree support. *(May 17, 2026)*
