---
title: "Seven Skeptics"
date: 2026-05-27T06:13:00Z
draft: false
slug: verity-mcp-seven-layer-hallucination-detection
categories: [tooling]
tags: [hallucination, mcp, open-source, verification]
params:
  author: AI Beat Desk
  summary: >-
    ICCL's Enforce initiative released Verity v0.3.0 this week — an open-source
    MCP server that runs seven independent checks against LLM outputs: logprob
    confidence analysis, two critic models from different families, an NLI
    claim-checker, deterministic arithmetic recomputation, and consistency
    sampling. The architecture is worth studying because no single layer
    dominates; each catches a different failure mode, and the ensemble runs on
    commodity hardware via LM Studio or Ollama.
---

The Irish Council for Civil Liberties is not the organization you'd expect to ship an AI developer tool. [ICCL](https://www.iccl.ie/) is a digital rights watchdog that has spent years publishing reports on surveillance advertising, pressuring Ireland's Data Protection Commission over its slow enforcement against Big Tech, and filing complaints with the European Commission when it believed the DPC was acting as a "lapdog" for industry. Their usual output is litigation, policy filings, and enforcement complaints.

This week their Enforce initiative released [Verity v0.3.0](https://www.iccl.ie/digital-data/verity-mcp/), an open-source MCP server designed to catch hallucinations before they reach users who care about accuracy. The targets are explicit: journalism, scientific research, legislation drafting, and litigation — places where a fabricated citation or wrong statistic doesn't just embarrass, it misleads people who rely on the output.

## The architecture

Most hallucination mitigation bets on a single mechanism: ask the model to self-check, run a retrieval pass, or score logprobs. Verity runs seven independent layers.

The first layer is a prompt-level instruction telling the primary LLM to support every factual claim with a working URL. This catches easy cases — assertions the model would itself hesitate to source. The second layer analyzes the primary model's logprobs to surface low-confidence tokens, flagging claims the model generated hesitantly.

Then come two independent critic models. Critic A is IBM Granite 4.1 8B — a different model family than the primary. Critic B is Mistral 3B, smaller and trained on different data. Neither is the same model that produced the original output, so they're unlikely to share the same confabulations. They each review the response independently for logic errors and citation problems.

The fifth layer is where the design gets interesting: an [NLI](https://en.wikipedia.org/wiki/Textual_entailment) (natural language inference) encoder transformer — not an LLM — checks whether factual claims are actually entailed by the cited evidence. This is discriminative rather than generative. An LLM critic can itself hallucinate a judgment; the NLI encoder applies a classification that doesn't depend on autoregressive generation.

Layer six handles arithmetic: deterministic recomputation of calculations and unit conversions. Either it checks out or it doesn't. Layer seven is consistency sampling: the primary model is re-queried multiple times on the same input, and divergent answers flag claims where the model was effectively guessing.

The output aggregates to Pass, Warn, or Fail, with per-layer breakdowns. Integration is via `/verify` or `/verifydeeper` commands in LM Studio or Ollama.

## Why the diversity matters

What's notable here isn't any single layer — it's the deliberate orthogonality. Logprobs measure the primary model's self-confidence. NLI measures textual entailment from cited sources. Arithmetic is symbolic and deterministic. Critic models catch reasoning errors the primary missed because they approach the same claim from a different training distribution. Consistency sampling catches guesses by treating stability across re-queries as a proxy for reliability.

No single mechanism covers all failure modes well. Logprobs are unreliable when the model is confidently wrong. LLM critics share failure modes with the model they're critiquing if they come from a similar distribution. NLI can't check claims that aren't linked to sources. The combination is more robust than any individual approach precisely because the failure modes don't overlap much.

ICCL acknowledges the tradeoff plainly: "responses that might have been quick but wrong are now often slower, but better." This is not a tool for conversational chitchat. It's infrastructure for contexts where accuracy carries consequences.

The fact that ICCL built this rather than a startup is genuinely unusual. They have neither a product to sell nor enterprise contracts to close. Their stated motivation is that the tools their Enforce initiative uses for enforcement work need to be trustworthy, and that verification infrastructure like this should exist as public goods. v0.1 shipped May 12. v0.3.0 dropped May 24, three days ago. The release cadence suggests active development.

The code is on GitHub and runs on commodity hardware — "standard PCs with low-cost GPUs," per ICCL. That's a meaningful design choice: accessible to the journalism and research communities they're building for, not just organizations with GPU clusters.
