---
title: "Three Passes Through the Same Weights"
date: 2026-11-23T07:00:00+00:00
draft: false
slug: inkling-2-mini
categories: [models]
tags: [open-weights, MoE, efficiency, inference]
params:
  author: AI Beat Desk
  summary: >-
    Thinking Machines Lab released Inkling 2 Mini on November 20, an open-weights
    26.4B-parameter MoE that loops its middle layers three times per forward pass.
    With only 3–5B active parameters and a 20 GB download, it reaches 79.2%
    SWE-bench Verified — a result that pushes on the question of whether more
    compute at inference beats more parameters in memory.
---

Seventeen days after releasing [Inkling 2 Small](https://ai-beat.github.io/news/2026/11/inkling-2-small/), Thinking Machines Lab put out [Inkling 2 Mini](https://thinkingmachines.ai/news/inkling-2-mini/) under Apache 2.0. The name is slightly misleading: at 26.4 billion total parameters, it is not particularly small. What makes it interesting is that only about 3.4 to 5.4 billion of those parameters are active per token, and the model achieves this efficiency not by routing to a sparse subset of experts — the usual MoE trick — but by passing through the same middle layers multiple times in a single forward pass.

The architecture has 24 layers. The two middle blocks each execute three times before the final layers run. That means each token experiences 48 layer applications even though the weight count is 26 billion. The company reports that looping the middle layers outperformed looping the last layers on 27 of 28 evaluation tasks. That is a surprisingly clean result, and it suggests the middle layers carry most of the representational work — a hypothesis that has been floated in mechanistic interpretability work but is rarely validated this directly.

The practical upshot is that you get a model that fits in about 20 GB and can run on an Apple Silicon Mac with 32 GB of RAM through Ollama, while delivering 79.2% on SWE-bench Verified. For comparison, [Inkling 2 Small](https://thinkingmachines.ai/news/inkling-2-small/) — a 594B MoE with 17B active parameters — is out of reach for most consumer hardware. Mini trades off factual recall (the company says it is weaker there than Small) and can get stuck in long reasoning loops, but for agentic coding on a laptop, the tradeoff seems designed rather than accidental.

It is worth comparing the two design choices Thinking Machines has made in the same month. Small used extreme sparsity (35× ratio between total and active parameters) to stay compute-efficient at scale. Mini uses layer looping to get more reasoning out of fewer stored weights. These are complementary strategies. Sparse MoE says: at training time and memory, pay for a large model; at inference, only activate a small slice. Loop-based models say: keep the weight count small; at inference, spend compute running the same weights repeatedly. Both arrive at the same goal — fewer active FLOPs per parameter stored — but through different mechanisms.

The company evaluates it on agentic traces up to 512K tokens, suggesting the context window is substantial. The model card includes an honest warning: it may take destructive actions without asking for confirmation. That caveat will not stop developers from using it, but it is the right thing to disclose for a model positioned at laptop-scale autonomous coding.

The [weights are on Hugging Face](https://huggingface.co/thinkingmachines/Inkling-2-Mini) and GGUF builds are available for llama.cpp. Ollama users on Macs with at least 32 GB of unified memory can pull it directly. The research variant exposes a knob to run one or two loop passes instead of three, trading response quality for speed — which is a useful handle for anyone trying to understand what the extra passes are actually buying.
