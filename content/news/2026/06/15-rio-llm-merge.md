---
title: "The Weights Don't Lie"
date: 2026-06-15T06:07:09+00:00
draft: false
slug: rio-llm-merge
categories: [open-source]
tags: [models, open-source, attribution, merging, governance]
params:
  author: AI Beat Desk
  summary: >-
    Rio de Janeiro's municipal AI company IplanRIO released Rio-3.5-Open-397B
    with claims of frontier performance, but an analysis of the open weights
    showed it is a simple 0.6/0.4 element-wise merge of Nex-N2_pro and
    Qwen3.5-397B-A17B. The model even introduces itself as Nex when
    the system prompt is removed. The episode illustrates the double-edged
    nature of open weights: the same transparency that enables community
    adoption also makes misrepresentation unusually easy to catch.
---

Municipal governments building AI models is not new. What is new, or at least newly visible, is a city government getting caught misrepresenting one.

Rio de Janeiro's municipal IT company, IplanRIO, released [Rio-3.5-Open-397B](https://huggingface.co/prefeitura-rio/Rio-3.5-Open-397B) on Hugging Face under an MIT license last week with a press push describing it as a "homegrown" model that rivaled ChatGPT and Claude and outperformed DeepSeek V4. The model is a 397-billion-parameter mixture-of-experts with 17 billion active parameters per token — a legitimate size for frontier-tier claims. The benchmarks it published looked credible on the surface.

Then someone looked at the weights.

A [GitHub issue on the Nex-AGI repository](https://github.com/nex-agi/Nex-N2/issues/4) showed that every weight tensor across all 60 layers of Rio-3.5-Open-397B carries the same 0.6/0.4 blend ratio — 60% Nex-N2_pro, 40% Qwen3.5-397B-A17B — consistent to thousands of standard deviations. This is not a training artifact or architectural coincidence. It is what a direct element-wise merge looks like. The operation is simple: take model A's weights, take model B's weights, compute \(w_{\text{result}} = 0.6 \cdot w_A + 0.4 \cdot w_B\) for every tensor. It requires no GPUs, no training data, and about as much engineering effort as a matrix addition.

The detail that made this difficult to dismiss was behavioral: strip Rio-3.5's system prompt and ask the model who it is. According to the analysis, it identifies itself as "Nex, from Nex-AGI" approximately 79% of the time, reciting Nex-AGI's backstory verbatim. The model was not trained on any Rio-specific identity — it just inherited Nex-N2_pro's self-description, because it *is* Nex-N2_pro at 60% weight.

IplanRIO's response, per community reports and a subsequent update to the Hugging Face model card, was to say they had accidentally uploaded the wrong version. The current model card now describes Rio-3.5 as "created by merging Nex-N2-Pro and Qwen3.5-397B-A17B, followed by on-policy distillation." The on-policy distillation is the additional fine-tuning work they claim distinguishes their actual model from the merge-only checkpoint that was initially shipped. Fine-tuning on top of a merge is real work, and the uploaded checkpoint may genuinely have been an intermediate artifact. But the disclosure of the merge basis only appeared *after* the community spotted it in the weights, not as part of the original release announcement.

Nex-AGI is credited in the Rio-3.5 model card, which is something. But the city's public communications did not describe the model as a post-trained merge of two existing open-source checkpoints — they described it as a homegrown system that their team had built. That framing is what generated the news coverage, and the coverage is what generated the scrutiny.

The parallel that came up immediately in the community discussion was Reflection 70B, the model from late 2024 that was initially promoted as a new state-of-the-art open model before analysis showed it was a fine-tuned version of Llama-3.1, with benchmark numbers that did not replicate independently. The pattern is similar: dramatic capability claims, open weights, community forensics, followed by a clarifying statement that reframes rather than retracts.

What's worth dwelling on is the structural irony. Rio released the weights under MIT, which is the right thing to do for a public institution — transparency, reproducibility, community utility. But the same act of releasing the weights made the misrepresentation immediately auditable. Closed-source models cannot be checked this way. When a company releases an API and publishes benchmark numbers, you can run your own evals, but you cannot inspect the weight tensors to see whether the model is what they claim. The open-source commitment that should have made Rio-3.5 more trustworthy is precisely what enabled the community to verify it was not what the press release said.

Nex-N2_pro is itself post-trained on Qwen3.5-397B-A17B under Apache 2.0, which permits derivative works and commercial use with attribution. The license is compatible. The attribution question is more about narrative than legal compliance: who built this, and what work did Rio actually do? A merge is a legitimate technique — model merging has produced genuinely useful artifacts — but it is different from training, and claiming you built something that you assembled from existing public checkpoints is a material misrepresentation when the distinction is not disclosed.

The weights told the truth. That's what they're for.
