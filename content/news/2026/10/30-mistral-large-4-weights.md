---
title: "Le Chonk Lands: Mistral Ships 1T-Parameter Open Weights"
date: 2026-10-30T06:15:00+00:00
draft: false
slug: mistral-large-4-open-weights
categories: [models]
tags: [open-source, mistral, moe, agentic, coding]
params:
  author: AI Beat Desk
  summary: >-
    Mistral just published the open weights for Mistral Large 4, a 1-trillion-parameter
    mixture-of-experts model with ~52B active parameters. It leads the open-weight
    field on coding agent benchmarks and is designed for sovereign, self-hosted
    deployment — completing Mistral's pattern of shipping frontier-quality models
    that you can actually run yourself.
---

Mistral has just [published the open weights](https://mistral.ai/news/mistral-large-4-oss/) for Mistral Large 4, internally nicknamed Le Chonk. It is a 1-trillion-parameter mixture-of-experts model — the largest freely downloadable LLM by parameter count — with around 52B active parameters per forward pass. The API preview went live earlier this month; the weights dropped today.

The benchmark numbers Mistral is leading with are agentic rather than the usual academic suite. On FrontierCode 2.0 it scores 19.7%, which the company says is the highest of any open-weight model. On Artificial Analysis's Coding Agent Index it ties GLM-5.3 at 48% while using 47% fewer tokens per task. Terminal-Bench 4.0, which tests multi-step CLI work, comes in at 41% — up 13 points from the preview and ahead of every other open model in the chart.

The emphasis on agentic tasks is deliberate. Mistral has positioned this model squarely at the deployment profile where a business runs a persistent, tool-using agent over long contexts in a constrained environment. The model supports three reasoning modes (Low, Medium, High) and is served via vLLM and SGLang on Hugging Face, OpenRouter, NVIDIA-hosted endpoints, and Azure Foundry. A reasonable reading is that Mistral is targeting the customers who were going to pay Anthropic or OpenAI for an API but would rather not share their data with an American company's infrastructure.

That framing is explicit in the announcement. Mistral describes the model as trained from scratch in Europe, and several government and security bodies — ENISA among them — are listed as evaluating it. The cybersecurity benchmarks (Cyber Index: 51%, attack resistance: 96.6% on malicious request refusals) read as sales copy aimed at European defense and critical-infrastructure contracts.

The license is a modified MIT they're calling the Mistral Ecosystem Open-Weight license. Free to use, modify, and redistribute — including for commercial applications — as long as you're not offering it as a hosted Model-as-a-Service, for which you need a separate commercial agreement. This is the same architecture Llama adopted: open for self-hosting, licensed for resellers. Given Mistral's revenue depends on their own API, it makes structural sense.

A trillion parameters is genuinely a lot. Hardware requirements to serve this at speed are significant — the attention pattern from the MoE means you can load it on fewer GPUs than a dense model of the same size, but it is still not a commodity deployment. The pricing ($0.70/M input, $2.10/M output via Mistral's own API) is competitive with frontier closed models. If the benchmarks hold under independent evaluation, this moves the state of what you can self-host.

What stands out is the trajectory. Mistral started with small, sharp models and has scaled consistently while keeping open weights as the core product. They have not dropped a model that failed to compete meaningfully on at least one relevant benchmark. Le Chonk is their biggest bet, and the fact that it arrives with an agent-specific benchmark narrative rather than academic leaderboard positioning suggests they have done serious work on the deployment use case, not just the eval.

The weights are on [Hugging Face](https://mistral.ai/news/mistral-large-4-oss/). At 1T parameters, you will need real hardware. But you will own it.
