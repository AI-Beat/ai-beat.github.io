# AI Headlines — 2026-10-31

- [Show HN: A working Nintendo in ~1GB of Qwen weights (nesformer)](https://github.com/dbreunig/nesformer) — An NES emulator whose 6502 CPU is analytically "compiled" into the weights of a DeltaNet-based model (~466M params, 9.3M nonzero); no gradient descent involved, just weight engineering — DeltaNet state rows become addressable memory cells. Runs Super Mario Bros. and Thwaite. *(October 30–31, 2026)*

- [Backdoor in Claude Code for VS Code Extension on the Cursor Marketplace](https://phorion.io/blog/offical-claude-code-vsx-compromise/) — Phorion reports versions 2.1.312 and 2.1.313 of the Claude Code VS Code extension were trojanized in the Cursor Marketplace; the payload adds an attacker SSH key, installs a reverse tunnel, and persists as a fake com.apple.sysmond LaunchAgent. Cursor has since removed both versions. *(October 30–31, 2026)*

- [The Sleep Framework: A New Way for LLMs to Retain Knowledge in Continual Learning](https://research.google/blog/the-sleep-framework-a-new-way-for-llms-to-retain-knowledge-in-continual-learning/) — Google Research introduces a wake/sleep training loop where a deployed LLM consolidates new knowledge into low-rank MoE layers without forgetting prior skills; reports near-zero forgetting on rare-language translation benchmarks and near-perfect scaling to 10M tokens on BABILong. *(October 27, 2026)*
