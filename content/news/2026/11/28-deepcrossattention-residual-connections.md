---
title: "Layers Talking to Layers"
date: 2026-11-28T06:12:06+00:00
draft: false
slug: deepcrossattention-residual-connections
categories: [training]
tags: [google, transformers, architecture, attention, residual-connections, training-efficiency]
params:
  author: AI Beat Desk
  summary: >-
    Google Research's DeepCrossAttention, originally presented at ICML 2025, got a
    detailed blog post this week. The core idea is to apply attention across layers
    rather than across tokens, letting each layer selectively pull from the entire
    earlier stack rather than receiving a simple sum. The result: comparable model
    quality in up to five times fewer training steps, with less than 0.2% added
    parameters — and it can be retrofitted into an existing pretrained model.
---

Standard transformer residual connections are a fixed recipe: add the output of each layer to its input, with equal weight, every time. It works remarkably well, and the field has largely left it alone. A [paper from Google Research](https://research.google/blog/supercharging-transformer-residual-connections/), presented at ICML 2025 and covered in a detailed blog post this week, asks what happens if you replace that addition with something that actually learns which prior layers to pull from.

The answer is DeepCrossAttention (DCA), and the training efficiency numbers are striking.

The standard picture of a deep transformer is a chain: each layer reads the residual stream it receives, transforms it, and adds its output back. The model at layer 32 has only indirect access to what layer 3 thought; that signal has been summed and normalized many times over. DCA changes the residual read: instead of accepting a straight sum, each layer runs cross-attention across the entire stack of prior layer outputs. The token at position \(i\) in layer \(l\) can query the embeddings of that same token at every earlier layer, attending with learned weights that are partly input-independent (a base bias) and partly computed from the current token representation. This is depth-wise attention — across layers, not across the sequence.

The computational overhead is modest. The mixing weights are computed with a small projection, the attention is applied only over the depth dimension (not the full sequence), and the whole mechanism adds less than 0.2% to the parameter count. The initialization is set so DCA starts behaving exactly like a standard residual connection, so it can be dropped into a pretrained model without destabilizing it.

The benchmarks are where this gets interesting. On the LM1B language modeling benchmark, DCA reaches a perplexity below 14 at 30 layers; a standard transformer needs 42. On C4, the authors report comparable quality in up to five times fewer training steps. When they retrofitted DCA into an already-trained model and continued training, it converged faster than continued standard training would have.

Why does this work? The intuition is that deep networks have a representational bottleneck in their lower layers: early-layer features that might be useful much later get progressively diluted as they're mixed with higher-level representations at each residual step. The mixture-of-experts literature addresses this horizontally, routing different tokens to different sublayers. DCA addresses it vertically, giving later layers a way to bypass the dilution and query earlier representations directly. The learned base weights turn out to implement something like a soft skip connection pattern — the network discovers for itself which layer combinations are useful.

The paper also cites Moonshot AI's attention residuals work as a parallel line of investigation, so this idea is apparently surfacing in multiple groups at roughly the same time. That's usually a sign that something is genuinely useful rather than a one-off result.

The practical ceiling for this technique isn't clear yet. The reported experiments are on language modeling benchmarks with reasonably-sized models, not at frontier scale. Whether the training speedup holds as models get larger and more heavily parallelized across devices is an open question. The retrofit capability is probably the most immediately useful property: if you can drop DCA into a model that's already converging and get a meaningful boost from the remaining compute, that's a straightforward cost savings with low risk.
