# AI Headlines — 2026-08-03

- [Explorative Modeling: Unlocking a Third Pretraining Axis and End-to-End Generation](https://arxiv.org/abs/2607.27372) — Alexi Gladstone proposes adding a best-of-K selection step to the generative pretraining loop, training only on the candidate closest to the target; achieves 4.1× better FLOP efficiency and 6.2× better sample efficiency on image models, with scaling gains that grow with both model and data size, and modest but promising improvements on autoregressive language models. *(July 29, 2026)*

- [How we set up our cloud agent environment](https://cursor.com/blog/cloud-agent-environment) — Cursor describes the infrastructure work behind their cloud agent deployment: a Ubuntu Dockerfile to match Mac dev environments, an `anydev` CLI that abstracts build commands, and Cloud Doctor (an automation that diagnoses agent environment failures and opens PRs to fix them); result: cloud agents went from roughly one in ten merged PRs in December 2025 to more than half today. *(July 30, 2026)*

- [Qwen3.8-Max: A New Bar for Coding and Cowork](https://qwen.ai/blog?id=qwen3.8) — Alibaba's 2.4T-parameter flagship model launches in preview access on Qoder and QoderWork, claiming benchmark parity with Anthropic's Fable 5 based on internal evals; no third-party independent verification yet and no open-weight release date announced. *(August 3, 2026)*
