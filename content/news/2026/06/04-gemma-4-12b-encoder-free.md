---
title: "Gemma 4 12B Goes Encoder-Free"
date: 2026-06-04T06:11:27+00:00
draft: false
slug: gemma-4-12b-encoder-free
categories: [models]
tags: [models, multimodal, open-weights, google, architecture, audio]
params:
  author: AI Beat Desk
  summary: >-
    Google DeepMind's Gemma 4 12B discards the conventional encoder-stack approach
    to multimodal models, feeding raw pixel patches and audio waveforms directly
    into the LLM backbone through lightweight linear projections. The result fits
    in 16 GB of RAM, accepts native audio, and fine-tunes as a single unified model.
---

The standard formula for adding vision to a language model is well established: attach a pretrained vision encoder (usually CLIP, SigLIP, or an in-house variant), project its output into the language model's residual stream, and optionally freeze the encoder during early fine-tuning. Extend the same pattern to audio with a Whisper-style encoder and you have the architecture underlying most of the multimodal models released over the past two years.

[Gemma 4 12B](https://blog.google/innovation-and-ai/technology/developers-tools/introducing-gemma-4-12b/), released June 3 by Google DeepMind, removes all of that. There is no vision encoder. There is no audio encoder. Instead, vision inputs go through a 35M-parameter embedder — a single matrix multiplication from 48×48 pixel patches directly to the LLM's hidden dimension, plus positional embeddings and layer norm. Audio inputs are handled by slicing raw 16 kHz waveforms into 40ms frames (640 floats each) and projecting them linearly into the same space as text tokens.

The 35M vision embedder represents roughly 0.3% of the total parameter count. The audio projection is lighter still. You are not adding a second model — you are adding a doormat.

## Why This Is Interesting

The appeal of separate encoders is that they arrive pretrained on large vision-language contrastive datasets, carrying semantic understanding of images before the language model sees any of it. CLIP knows that a photo of a golden retriever looks like the phrase "golden retriever." A linear projection from raw pixel patches does not start with that knowledge; the language model backbone has to compensate.

The argument that this can work anyway has been gaining traction. The backbone in a 12B parameter model trained on diverse multimodal data is powerful enough to develop visual understanding on its own, provided the input projection is stable enough that the backbone can learn through it. The unified training setup helps here: because there are no frozen components, gradients flow through everything simultaneously.

The practical benefit that is harder to argue with: fine-tuning becomes a single pass. You run LoRA (or full fine-tuning) on one model, and your improvements apply jointly across text, vision, and audio. With the encoder-stack approach, there is an ongoing question of how much to update the encoder vs. the adapter vs. the backbone, and these choices interact in ways that require careful tuning. Gemma 4 12B eliminates the interaction.

## Numbers and Limits

The benchmarks: MMLU Pro 77.2%, GPQA Diamond 78.8%, LiveCodeBench v6 72.0%, MMMU Pro (vision reasoning) 69.1%, Tau2 agentic average 69.0%, FLEURS audio word error rate 0.069. Those numbers place it "close to the 26B active-MoE line" on most tasks according to the developer guide, which is a reasonable claim. There are no direct controlled comparisons between this architecture and encoder-based models at equivalent parameter counts, so how much is gained or lost by removing the encoder is not directly observable from the release materials.

Context window is 256K tokens. Audio clips up to 30 seconds, video up to 60 seconds at roughly 1 FPS. A Multi-Token Prediction (MTP) draft model is bundled for speculative decoding. Full BF16 weights are 26.7 GB; Q4 weights come in around 6.7 GB, fitting comfortably alongside a KV cache in 16 GB of unified memory.

Native audio at this weight class is the most notable capability extension. Most competing open-weights models in the 12–15B range — Qwen 2.5 VL, Llama 4 Scout, Mistral's vision variants — either lack audio entirely or require a separate audio encoder at inference time that substantially increases the memory footprint. Gemma 4 12B handles audio inside the same model, with no external dependency.

Apache 2.0 license. Available on [Hugging Face](https://huggingface.co/google/gemma-4-12b-it) and Kaggle.

The open question is whether the minimal embedder design holds up on inputs that require fine-grained visual understanding — dense text in images, complex diagrams, close-up detail where the 48×48 patch grid resolution matters. The MMMU Pro score at 69.1% suggests it is competitive on structured multimodal reasoning. Whether it loses ground to encoder-based models in more demanding visual tasks is worth watching as community evals come in over the next few weeks.
