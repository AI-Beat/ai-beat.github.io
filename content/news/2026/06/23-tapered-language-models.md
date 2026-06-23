---
title: "Give Early Layers More"
date: 2026-06-23T06:10:00+00:00
draft: false
slug: tapered-language-models
categories: [research]
tags: [architecture, transformers, mlp, efficiency, pretraining]
params:
  author: AI Beat Desk
  summary: >-
    A paper submitted yesterday finds that reducing MLP width monotonically from
    early to late transformer layers — using a cosine schedule — consistently improves
    performance across three scales and four architectures at zero additional cost.
    Later layers refine the residual stream rather than transform it, so the standard
    uniform allocation gives too much capacity to the wrong end of the network.
---

Every layer in a standard transformer uses the same MLP width. That's not a principled design decision — it's a historical default, chosen because it's simple and because uniform allocation made the early architecture literature easier to describe. [A paper submitted yesterday to arXiv](https://arxiv.org/abs/2606.23670) provides systematic evidence that it's also suboptimal, and that you can improve on it without changing your parameter budget.

The proposal is called Tapered Language Models. The mechanism: instead of giving every feedforward network the same width, use a smooth cosine schedule that reduces MLP width monotonically from the first layer to the last, while keeping the total parameter count constant. Earlier layers get more capacity; later layers get less. The total is redistributed, not increased.

Across three model scales and four architectures — standard Transformer, Gated Attention, Hope-attention, and Titans — the tapered distribution consistently improves perplexity and downstream task performance over the uniform baseline. The reverse allocation, giving more parameters to late layers than early ones, hurts performance. The direction matters; the improvement isn't symmetric.

## Why the direction matters

The mechanistic interpretability literature has spent several years characterizing what different layers in a transformer actually do. The emerging picture is that early layers do the heavy lifting: building factual associations, establishing syntactic structure, constructing basic semantic representations. Later layers do something more like refinement — adjusting outputs for context, formatting responses, resolving surface ambiguities in the residual stream. The earlier stages are doing fundamental transformation; the later stages are modifying an already-useful representation at the margin.

If that characterization is roughly right, uniform capacity allocation is a mismatch. The layers doing the harder work get the same budget as the layers polishing the output. Shifting capacity toward earlier layers addresses this mismatch directly.

The cosine schedule is the practical operationalization. It provides a smooth gradient of width across the network, avoiding sharp discontinuities in representational capacity that could disrupt information flow across layers. The total parameter budget stays fixed — the width at any layer is determined by its position in the depth schedule.

## What makes the result credible

The generalization across four architectures is the most interesting aspect. Titans uses a memory-based attention mechanism quite different from standard softmax attention. The fact that the tapering improvement appears there as well as in conventional transformers suggests the finding isn't tied to a quirk of any single design. It appears to be something about the depth structure of residual networks more broadly.

This also connects to findings in the network pruning literature. Late layers in transformers tend to have more redundant attention heads than early layers, and can often be pruned more aggressively with less quality loss. The TLM result is a softer, prospective version of the same insight: don't build in the redundancy to begin with, just allocate less to the layers that need less.

The paper was posted yesterday and hasn't yet had time to circulate widely. Whether the tapering principle has been independently discovered and applied in any of the large-scale training runs happening now is unknown — these things aren't always published. But the finding is cheap to implement, architecture-agnostic, and reproducible from the paper. The history of architectural improvements in this space suggests that a clean, zero-cost performance improvement tends to get absorbed into practice quickly. The default assumption that all layers deserve equal parameters probably has a limited remaining lifespan.
