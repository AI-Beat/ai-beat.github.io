---
title: "Byteification: Retrofitting Tokenized Models to Read Raw Bytes"
date: 2026-10-11T06:10:49Z
draft: false
slug: bolmo-byteification
categories: [research]
tags: [LLM, tokenization, open-weights, Ai2, multilinguality]
params:
  author: AI Beat Desk
  summary: >-
    A Nature paper from Ai2, LMU Munich, Cambridge, and UW describes how to
    convert a trained subword model into a byte-level one using less than 1%
    of a typical pretraining budget. The resulting Bolmo, Bwen, and Blama
    families retain most of the original capability while gaining the
    structural properties that tokenization normally destroys.
---

Subword tokenization is a compromise. It keeps sequence lengths manageable and lets models share learned representations across morphologically related words. But it also discards structure: the model never sees individual characters, so it can't reliably count letters, spell backwards, or handle strings that fall outside its vocabulary. For code and biological sequences the problem is more fundamental — bytes carry meaning at a level tokenization throws away.

The usual answer to this has been to train byte-level models from scratch. MegaByte, BLT, and similar work have shown this is possible but expensive, and the resulting models have tended to lag behind similarly-sized subword models on standard benchmarks.

A [paper published in Nature this week](https://www.nature.com/articles/s41586-026-11111-4) from researchers at Ai2, LMU Munich, Cambridge, UW, and Imperial College London takes a different approach: take a model that already knows things, and change how it reads. They call the process "byteification."

The technique works in two stages. In the first, the byte-level input components are trained while the rest of the model is frozen, letting the new front-end learn to produce embeddings the pretrained transformer already understands. This produces a stage-1 checkpoint that researchers can start from without repeating the full process. In the second stage, the whole model is fine-tuned jointly. The total training budget the paper reports is around 49 billion tokens — less than 1% of a typical large pretraining run.

The resulting models — Bolmo 1B and 7B (from OLMo 3), Bwen 8B (from Qwen 3 8B), and Blama 8B (from Llama 3 8B) — retain most of the capabilities of their source models while gaining the structural properties that byte-level processing provides: fine-grained character handling, no out-of-vocabulary tokens, and a vocabulary-free architecture that's consistent across languages and scripts. Bwen 8B is described as the strongest of the family, outperforming Bolmo 7B on Ai2's evaluation suite. On STEM tasks, the byteified models substantially outperform BLT 7B, the main byte-level baseline trained from scratch.

The practical implication is that the choice between "capable subword model" and "byte-level model with good structural properties" may not be as binary as it has looked. If byteification generalizes well across model families — and the fact that it works on OLMo, Qwen, and Llama suggests it might — then there's a path to byte-level models that doesn't require abandoning everything learned during large pretraining runs.

One thing worth watching: Ai2 has released stage-1 checkpoints on Hugging Face, which means the community can start building on the frozen-backbone intermediate without redoing even the cheaper second-stage training. Early follow-up work is already exploring capability transfer between byte-level models and efficiency tradeoffs in hierarchical byte architectures. The research direction has momentum in a way it didn't when byte-level models had to be trained from nothing.

The obvious application is code — compilers, interpreters, and binary analysis tools work on bytes and don't care about word boundaries. Biological sequences are another, since amino acid and nucleotide strings have their own structure that tokenization tends to smear. The multilingual case is perhaps the most straightforwardly useful: a byte-level model trained on one language's data doesn't need a separate tokenizer or special vocabulary coverage to handle another.

None of this lands as a sudden capability jump. But it's a clean result that removes a practical obstacle — the cost of getting to byte-level — and does it in a way that's reproducible starting from publicly available models.
