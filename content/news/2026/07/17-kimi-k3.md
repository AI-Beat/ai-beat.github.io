---
title: "Two Point Eight Trillion"
date: 2026-07-17T06:08:00+00:00
draft: false
slug: kimi-k3-open-frontier
categories: [models]
tags: [kimi, moonshot, moe, open-weight, multimodal, mxfp4]
params:
  author: AI Beat Desk
  summary: >-
    Moonshot AI announced Kimi K3 on July 16, claiming "the world's first open 3T-class
    model" at 2.8 trillion total parameters — with weights delayed until July 27. The
    architecture uses a 16-of-896 expert MoE with Kimi Delta Attention and MXFP4
    quantization-aware training, keeping active inference cost near a 50B model while
    scaling total capacity nearly three-fold over K2.
---

[Kimi K3](https://www.kimi.com/blog/kimi-k3) is Moonshot AI's latest model and, by their own framing, the first publicly announced open-weight model in the 3-trillion-parameter class. The numbers: 2.8 trillion total parameters, 16 experts active out of 896 per forward pass, a 1-million-token context window, and native multimodal support. Weights are promised by July 27 — ten days out from the announcement.

That last detail is worth sitting with. The "open" in "open frontier intelligence" currently means open access via API, not downloadable weights. You can call K3 today through Kimi's API and products (Kimi.com, Kimi Work, Kimi Code); you cannot yet run or inspect the model yourself. This isn't unusual for large releases, but it does mean "open" is doing some marketing work alongside its technical meaning.

The mixture-of-experts architecture is where the interesting engineering lives. With 16 of 896 experts active per token, and 2.8T total parameters, each expert holds roughly 3.1 billion parameters. Active inference draws on about 50 billion of them per forward pass, plus the non-expert transformer layers. That puts K3's inference cost closer to a 50B dense model than its headline figure suggests — relevant for anyone thinking about serving costs. The total parameter count determines what the model can know; the active count determines what you pay per token. The distinction between 2.8T and ~50B is exactly this: capacity versus compute.

Two architectural innovations are named in the announcement: Kimi Delta Attention (KDA) and Attention Residuals (AttnRes). The technical report is promised alongside the weight release, so neither gets a precise definition yet. "Delta" in attention contexts typically refers to mechanisms inspired by the delta rule in linear recurrent systems — allowing more efficient handling of long sequences by carrying a compressed state rather than attending over the full context history. Attention Residuals likely adds explicit skip connections through the attention sublayer, improving gradient flow in very deep architectures. Whether these descriptions match what the paper actually describes will be clearer on July 27.

Training used MXFP4 weights and MXFP8 activations — the Microscaling floating-point format formalized by Intel, AMD, Microsoft, and Nvidia for quantization-aware training. Training in MXFP4 rather than post-training quantizing down from BF16 is meaningfully different: the model learns to operate robustly at low precision from the start rather than being degraded after the fact. At 2.8T parameters, the storage implications are significant: MXFP4 halves weight storage compared to INT8 and is a quarter the size of BF16. Distributing model weights at this scale requires that efficiency to be practical.

Moonshot claims "2.5× improvement in overall scaling efficiency compared to Kimi K2." K2 was roughly a 1T parameter model; K3 at 2.8T is nearly three times larger in total parameters. If the 2.5× efficiency claim holds — compute-per-quality-unit on their benchmark suite — the effective capability jump is larger than the parameter ratio suggests. [Kimi K2.7 Code](https://ai-beat.github.io/news/2026/07/kimi-k2-7-copilot-open-weight/), covered earlier this month for its GitHub Copilot availability, was a coding-specialized branch of that lineage. K3 is positioned as the general frontier successor.

The timing is interesting alongside yesterday's [Inkling release](https://ai-beat.github.io/news/2026/07/inkling-thinking-machines/) from Thinking Machines Lab: 975B total / 41B active versus K3's 2.8T total / ~50B active. Both MoE models end up with similar active parameter counts per inference step — the inference cost difference between them may be smaller than the headline parameter gap implies. The key difference is the depth of the expert pool: K3's 896 experts offer the model a much larger set of specialized representations to route through, even if only 16 are active at once. More experts means more potential specialization, at the cost of routing overhead and harder training dynamics.

Whether any of this matters depends on what the weights reveal when they land. Benchmark numbers from a model provider's own evaluation suite carry the usual producer-benchmark caveats. The architecture claims (KDA, AttnRes, Stable LatentMoE) and the MXFP4 training approach are the details worth reading carefully in the technical report. Until July 27, K3 is a well-specified API and a set of claims. That's not nothing — the API is real and the routing is running — but the "open" half of the open-frontier pitch is still ten days away.
