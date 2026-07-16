---
title: "Thinking Machines Ships Inkling"
date: 2026-07-16T06:10:24+00:00
draft: false
slug: inkling-thinking-machines
categories: [models]
tags: [open-weights, moe, multimodal, training, mira-murati]
params:
  author: AI Beat Desk
  summary: >-
    Thinking Machines Lab, the startup founded by former OpenAI CTO Mira Murati,
    released its first public model on July 15: Inkling, a 975B total / 41B active
    mixture-of-experts trained on 45 trillion multimodal tokens, Apache 2.0 licensed,
    with AIME 2026 97.1% and SWEBench Verified 77.6%. The lab's explicit framing is
    "not the best, but the most customizable" — a positioning bet that the open-weights
    market rewards fine-tuning infrastructure over raw benchmark supremacy.
---

After roughly eighteen months of building mostly out of view, Thinking Machines Lab has shipped its first public model. [Inkling](https://thinkingmachines.ai/news/introducing-inkling/) is a mixture-of-experts transformer with 975 billion total parameters, 41 billion active per forward pass, trained on 45 trillion tokens of text, images, audio, and video. The weights are available on Hugging Face under Apache 2.0, which is about as permissive as open-weights licensing gets.

The architecture has some interesting choices. Each layer uses 256 routed experts with 6 active per token, plus a small set of shared experts that always run — the shared layer prevents the catastrophic forgetting that pure routing can cause when the model encounters rare token types. Attention is interleaved: sliding-window local layers alternate with global attention layers, which keeps memory linear for long contexts while preserving cross-document coherence. Short convolutions run alongside the attention mechanism for efficiency. The context window goes to 1M tokens. A smaller companion, [Inkling-Small](https://thinkingmachines.ai/model-card/inkling/), uses 276B total / 12B active and is released in preview; the team says it matches the larger model on many benchmarks at lower latency.

Training used Muon for the large weight matrices and Adam for smaller parameters. Muon — a momentum-based optimizer derived from Nesterov momentum and orthogonalization — has been showing up in recent large-scale runs because it can converge in fewer steps than Adam on matrices where the gradient has strong directional structure, which matters when you're processing billions of expert parameters. Post-training ran 30 million rollouts of asynchronous RL, a scale that suggests a sustained multi-cluster RL loop rather than a bounded RLHF fine-tune.

The benchmark numbers are strong. AIME 2026 at 97.1% puts it in the top tier of public reasoning evaluations. GPQA Diamond at 87.2% and SWEBench Verified at 77.6% round out a picture of a model that's genuinely capable across math, science, and software tasks. Safety evaluations show 98.6% on StrongREJECT and 78.0% on the FORTRESS adversarial suite.

What's notable is how the lab frames the release. The announcement is explicit: Inkling is "not the strongest overall model available today, open or closed." That's an unusual thing to lead with. The pitch is instead about fine-tuning surface — the model is designed to be a customizable foundation for domain-specific work, and Thinking Machines offers their Tinker platform to do that fine-tuning on top of it (at a 50% introductory discount). One technical acknowledgment from TechCrunch: early post-training data used outputs from [Kimi K2.5](https://github.com/MoonshotAI/Kimi-K2), though the team says the next version will use a fully self-contained pipeline.

Whether this positioning actually works is an open question. The open-weights field is increasingly crowded — [DeepSeek V4 Pro](https://github.com/deepseek-ai/DeepSeek-V4), [GLM-5.2](https://github.com/THUDM/GLM-4), [MiniMax M3](https://github.com/MiniMaxAI/MiniMax-M3) all compete at this tier. "Best for customization" is a harder story to tell than "best on the benchmark." But Apache 2.0 on a 41B-active MoE with 1M context, multimodal from the start, and inference support across [SGLang](https://github.com/sgl-project/sglang), [vLLM](https://github.com/vllm-project/vllm), TogetherAI, Fireworks, Modal, and Databricks is a real release, not a teaser. The practical barrier to running or fine-tuning it is low. We'll know soon enough whether the customization thesis holds up against labs with bigger compute budgets.
