---
title: "The 179x Tax on Non-English AI"
date: 2026-11-17T06:30:00+00:00
draft: false
slug: language-energy-divide-multilingual
categories: [research]
tags: [efficiency, multilingual, energy, inference, equity, EMNLP]
params:
  author: AI Beat Desk
  summary: >-
    An EMNLP 2026 paper measured total inference energy across eight languages
    and found a 179x gap between English and Pashto for identical request sets.
    The penalty has two compounding causes — per-token energy cost up to 8.3x
    higher in complex-script languages, and many more tokens needed for
    low-resource ones — and both directions also correlate with lower task
    accuracy.
---

If you benchmark LLMs only in English, you miss more than half the story. A team at the University of Michigan measured what it actually costs to run inference in multiple languages and published the results in the [EMNLP 2026 main proceedings](https://arxiv.org/abs/2606.21869). The gap is larger than most people working in this area have been assuming.

For a fixed set of requests, English consumed 17.6 kJ. Pashto consumed 3,147 kJ — roughly 179 times as much. This is not a quirk of one model or one hardware setup. The authors ran the [ML.Energy framework](https://ml.energy) across multiple models, hardware configurations, and task types, and found the disparity persists throughout.

Two distinct mechanisms compound to produce the gap. First, per-token energy cost varies by up to 8.3x across languages. Complex or rare scripts require the model to perform more computation per token — longer attention patterns, denser softmax operations, more character-level subword processing. Second, low-resource languages are more wastefully tokenized. A tokenizer trained primarily on English-language text needs more tokens to represent the same semantic content in Hindi, Bengali, or Pashto, so the total request length grows even before inference begins. Both penalties apply simultaneously to the worst-affected languages.

The accuracy side of this is the part that should attract attention. The languages with the highest energy costs also tend to have the lowest task accuracy. This is the "double penalty" framing the authors use: users who happen to work in Pashto pay more energy for worse answers. Neither problem is accidental — both trace back to training data distributions. High-resource languages get better tokenization, more training data, and consequently lower inference costs and higher accuracy. The correlation between resource availability and cost efficiency is not coincidental; it is structural.

There are roughly 5.2 billion people whose first language requires more than twice the energy of English speakers to produce equivalent output. At current scale this is not primarily a bill paid by individuals — cloud providers absorb it in operational costs — but it does shape what is economical to offer, what gets optimized, and where infrastructure investment flows. A language that is expensive to serve on commodity hardware does not get the same local deployment attention as one that runs efficiently on consumer GPUs.

The paper's recommendations are modest and practical: treat energy as a first-class metric in evaluations, include it in model cards, and report it alongside accuracy. That is a low bar, but it is not currently standard. The more structural fix — better multilingual tokenizers and more balanced training corpora — is slower work that is already underway in several open-source projects, though progress is uneven.

The measurement methodology here is clean enough to be reused directly. If you're building or deploying a multilingual system, the ML.Energy tooling is worth running on your actual traffic before assuming English benchmarks tell you anything about operational costs at scale.
