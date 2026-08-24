# AI Headlines — 2026-08-24

- [Decoding silent reading from non-invasive EEG](https://arxiv.org/abs/2608.20186) — Marquardt, Alchanat & Jain train a CNN–transformer with a CLIP-style contrastive objective against LLM word embeddings on 49 hours of 19-channel dry-electrode EEG; above-chance open-vocabulary word decoding with accuracy scaling log-linearly with data and no saturation in sight. *(August 20, 2026)*

- [Ray 2.58.0: KV-cache-aware routing for LLM serving](https://github.com/ray-project/ray/releases/tag/ray-2.58.0) — Tokenization moves from engine to the LLMRouter ingress replica, tokens are transmitted out-of-band, KV lifecycle events broadcast to all routers, and CPU-offloaded cache blocks now count toward prefix-match scoring — completing the KV-aware routing work previewed in 2.57. *(August 23, 2026)*

- [My agent.md to improve LLM-assisted code quality](https://fabiensanglard.net/agent.md/index.html) — Fabien Sanglard describes a persistent style-guide file that coding harnesses load on every session: keep functions short, early returns over nesting, enums over boolean params, layered architecture — and ask the agent to reload it when output quality drifts. *(August 21, 2026)*
