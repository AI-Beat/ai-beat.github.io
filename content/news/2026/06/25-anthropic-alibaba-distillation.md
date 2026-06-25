---
title: "28.8 Million Prompts"
date: 2026-06-25T06:04:00+00:00
draft: false
slug: anthropic-alibaba-distillation-attack
categories: [security]
tags: [security, policy, anthropic, distillation, export-controls]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic disclosed to the US Senate that operators affiliated with Alibaba
    ran 28.8 million exchanges against Claude through 25,000 fraudulent accounts
    over six weeks — the largest known distillation attack against Anthropic.
    The numbers are real; the framing is lobbying.
---

When Anthropic sent a letter to the Senate Committee on Banking, Housing, and Urban Affairs this week, they put a specific number on something the AI industry has mostly talked around: [28.8 million exchanges](https://www.cnbc.com/2026/06/24/anthropic-alibaba-distillation-campaign.html), approximately 25,000 fraudulent accounts, running from April 22 through June 5. That's what Anthropic describes as a distillation attack by operators affiliated with Alibaba and its AI research arm — "the largest known distillation attack on Anthropic to date."

"Adversarial distillation" is the term Anthropic uses, though the technique is just model distillation run against someone else's API without permission. The mechanics are straightforward: you systematically prompt a high-capability model to generate outputs — reasoning chains, code, explanations — and use those outputs as training data for a model you're building. Done carefully, you can train your model to approximate the behavior of the teacher without ever touching the teacher's weights. This is why model distillation has legitimate uses everywhere (smaller models trained on larger models' outputs are standard practice in ML), and also why the adversarial version is a meaningful threat: it converts API access into a labeled training corpus.

The scale is what distinguishes this from normal competitive intelligence. At 28.8M exchanges, you're not probing the model's behavior — you're running an industrial data collection pipeline. The 25,000 fraudulent accounts suggest systematic effort to spread load across rate limits and detection thresholds. This is organized infrastructure, not opportunistic scraping.

Whether it actually works — that's the more interesting question, and one where the answer is genuinely uncertain. You can absolutely train a model on another model's outputs and get something that behaves similarly in distribution; that's what student-teacher distillation has always done. But frontier model capabilities don't arise purely from output distributions. They emerge from the interaction of scale, curated training data, synthetic data pipelines, and RLHF choices that are not visible in API responses. Harvesting 28.8M Claude responses gets you Claude's outputs, not Claude's reward model, not the specific data mix that shaped its behavior at the margins, not the compute-dependent emergent capabilities that appear in ways that are hard to predict or replicate. Whether that's enough to materially close the gap between Alibaba's Qianwen models and Anthropic's Fable 5 is genuinely unclear — and Anthropic has obvious incentives to overstate the risk when writing to a Senate committee.

That context matters when reading this disclosure. Anthropic is lobbying. The letter comes at a moment when the Trump administration has already ordered Anthropic to [suspend access to Fable 5 and Mythos 5 for all foreign nationals](/news/2026/06/fable-mythos-export-ban/) — a directive with awkward operational implications given that it covers foreign national Anthropic employees as well as overseas users. Anthropic's interest in framing this attack as a national security emergency rather than a terms-of-service violation is legible. The export control apparatus is already moving; this letter is an attempt to shape its direction.

None of that makes the attack less real. Systematically prompting a competitor's model with 25K fraudulent accounts over six weeks is a real operation, the accounts were fraudulent, and Alibaba affiliates apparently decided the value of Claude's outputs was worth the risk. The legal question of whether API outputs constitute "AI capabilities" in the sense that export control law intends is unresolved — and Anthropic is trying to establish that framing before the law catches up with the technology.

What the incident does reveal clearly is how inadequate the defensive tooling is. Rate limits and account verification are the primary defenses, and apparently 25,000 fraudulent accounts was sufficient to run this campaign for six weeks before detection or disclosure. The better your model, the more valuable the target on its outputs. If distillation attacks can materially transfer model capability — even partially — then every API call to a frontier model is also a potential training data point for a competitor. The technical defenses against this are immature, and the policy defenses depend on export control regimes that were not designed with model outputs in mind.

The number 28.8 million will probably get cited in AI policy discussions for years. Whether it represents a genuine theft of proprietary capability or a very expensive fine-tuning dataset remains to be seen.
