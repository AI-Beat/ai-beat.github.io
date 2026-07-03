---
title: "RL Post-Training Lives in the Middle"
date: 2026-07-03T06:15:00+00:00
draft: false
slug: single-layer-rl-training-middle-layers
categories: [research]
tags: [rl, post-training, transformers, fine-tuning, efficiency, qwen]
params:
  author: AI Beat Desk
  summary: >-
    A new paper finds that reinforcement learning gains in transformers
    concentrate almost entirely in a narrow band of middle layers. Training
    just one layer at roughly 40–60% network depth can match or exceed
    full-parameter RL fine-tuning. The finding challenges the assumption that
    all layers participate equally in post-training, and has practical
    implications for compute-efficient alignment.
---

Reinforcement learning post-training — the technique behind most of the reasoning and instruction-following gains seen in frontier models over the past year — apparently doesn't use most of the network. [A paper submitted to arXiv on July 1](https://arxiv.org/abs/2607.01232) finds that RL improvements concentrate tightly in a narrow band of middle transformer layers, and that training a single one of those layers in isolation can recover most — sometimes all — of the gain you'd get from updating every parameter in the model.

The researchers introduce a measurement they call "layer contribution": for each layer k, the fraction of full-RL-training performance recovered when only that layer is updated. The formula is straightforward — \(\mathcal{C}(k) = (S_k - S_\text{base}) / (S_\text{full} - S_\text{base})\) — where \(S_\text{base}\) is the pretrained baseline and \(S_\text{full}\) is performance after updating all parameters. A value above 1.0 means the single-layer result beats full training. A value below 0.5 means the layer barely moves the needle.

What they found: layer contributions vary by more than 4× across the network. On Qwen3-1.7B, the best layer (Layer 10) achieves a contribution of 1.14 — it exceeds full training. The worst layer (Layer 24) contributes 0.28. The pattern holds across seven models spanning Qwen3 and Qwen2.5 at various scales, three RL algorithms (GRPO, Dr. GRPO, GiGPO), and tasks ranging from mathematical reasoning to code generation to agentic decision-making. High-contribution layers cluster consistently between 40% and 60% of network depth. Input and output layers are largely bystanders.

The obvious hypothesis — that middle layers simply change more during full RL training, so they're doing more work — turns out to be wrong. The researchers specifically checked: there's no consistent correlation between a layer's weight magnitude change during full training and its layer contribution score. The middle layers matter more in isolation without necessarily dominating in raw parameter update magnitude. That's the genuinely puzzling part, and the paper is honest that it lacks a theoretical account of why.

The practical implication is clear even without the theory: if a single middle layer can match full-parameter RL training, you're spending most of your compute budget updating layers that contribute little. Three training strategies follow from the findings — boosting the learning rate on high-contribution layers, training only those layers, or using a heuristic that selects middle layers by position without needing to profile individual contributions first. All three outperformed standard uniform RL training in experiments.

The scope caveat is real: the validation is primarily on mathematical reasoning tasks. The authors note that extending the guided training strategies to coding and agentic tasks is left for future work. This matters because the community is most interested in applying RL to coding and long-horizon agent scenarios, where the distribution of relevant computation across layers might differ. Whether the finding generalizes cleanly to, say, a 70B+ MoE trained for tool use is an open question.

HN commenters raised a few good objections. One experienced RL practitioner pointed out that RL training is already fragile — reward model collapse, KL divergence explosion, out-of-distribution failure modes — and adding layer selection as another variable increases the surface area for things to go wrong. LoRA already handles parameter-efficient RL in practice, and it has years of operational experience behind it. That's a fair point; a research result on 1.7B to 7B models with clean math benchmarks doesn't immediately translate into "stop doing full-parameter RL on your production pipeline."

Still, the finding is worth sitting with. If it survives replication at larger scales and across more task distributions, it would have significant implications for how efficiently you can do RL post-training. The 40-60% depth clustering also maps interestingly onto what's become conventional wisdom about transformer layer roles: early layers handle token-level syntactic processing, middle layers perform more abstract semantic transformations, and late layers specialize in output vocabulary projection. RL training seems to touch the semantic transformation part most. That's not entirely surprising in retrospect — you're trying to adjust the model's reasoning patterns, not its vocabulary mapping — but having it quantified this precisely is new.

RadixArk released [Miles](https://pytorch.org/blog/miles-a-pytorch-native-stack-for-large-scale-llm-rl-post-training/) on June 30, a PyTorch-native framework for large-scale RL post-training that composes SGLang, Megatron-LM, and Ray into a unified pipeline with async rollout-training overlap and MoE-aware weight synchronization. The timing is coincidental, but the two pieces point in the same direction: the field is developing both a more precise mechanistic understanding of what RL post-training does inside the network, and better infrastructure for doing it at scale.
