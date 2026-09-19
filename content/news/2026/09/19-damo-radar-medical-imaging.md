---
title: "One Model, 146 Diagnoses"
date: 2026-09-19T05:57:00+00:00
draft: false
slug: damo-radar-medical-imaging
categories: [research]
tags: [research, open-source, healthcare, vision, multimodal]
params:
  author: AI Beat Desk
  summary: >-
    Alibaba's DAMO Academy released Damo Radar, a vision-language model covering
    146 abdominal conditions across 18 organs in contrast-enhanced CT, achieving
    AUC 0.913 across all findings in 40,000 real-world exams. The work appeared in
    *Science* and is fully open-sourced on GitHub and HuggingFace.
---

Medical AI has historically been narrow by design: one model, one disease, one imaging modality, one institution's dataset. That narrowness was partly a practical necessity—collecting enough labeled data to train anything broader was a research project in itself—and partly a regulatory strategy: a narrow scope makes validation tractable. [Alibaba's DAMO Academy](https://damo.alibaba.com/) just released a model that marks a meaningful step away from that pattern.

[Damo Radar](https://github.com/alibaba-damo-academy/damo-radar), open-sourced yesterday on GitHub and HuggingFace, is a vision-language model trained on contrast-enhanced CT scans. It covers 18 abdominal organs and identifies 146 clinical findings, including malignant tumors, benign lesions, and a range of other abnormalities. In a validation set of approximately 40,000 real-world examinations, it achieved an average AUC of 0.913 across all 146 findings. The results were published in *Science*.

The 0.913 AUC figure requires some unpacking to appreciate. Medical AI benchmarks often look impressive because the model is good at the few common findings that make up most of the training data, while rare conditions get averaged away. AUC across 146 conditions simultaneously, in a dataset of real clinical scans rather than curated research sets, is harder to game that way. The conditions span the full spectrum of abdominal pathology—common findings like liver lesions alongside rare conditions that most radiologists encounter infrequently. Holding 0.913 across that range is a different achievement from holding 0.95 on a binary cancer/no-cancer classifier.

The comparison to radiologists—the paper's headline claim that the model "outperformed most radiologists"—needs to be read with the study's specifics in hand. Radiologist performance varies considerably depending on subspecialty training, case mix, and reading conditions, and "most radiologists" is a distribution, not a fixed threshold. The *Science* publication means the methodology was reviewed rigorously; the details of that comparison are worth reading before drawing strong conclusions about clinical deployment readiness.

What matters more for practical purposes: the scope. Abdominal CT interpretation is one of the highest-volume imaging tasks in clinical medicine. Radiologists read abdominal CTs under time pressure across the full range of pathology, and the cognitive load of tracking 146 possible findings simultaneously is real. A model that reliably identifies findings a radiologist might miss under volume pressure, or flags rare conditions that fall outside a generalist's regular pattern recognition, has clear utility even without beating radiologists in aggregate. The use case isn't replacement—it's a second pass that catches what the first pass missed.

The open-source release includes code, training framework, and weights. Releasing the full training framework alongside a *Science*-published result is an unusually transparent approach for a major tech company's applied research group, and it means academic medical centers can actually adapt the model to their local imaging protocols and patient populations rather than treating it as a black box.

The coverage so far has focused on 150 conditions and the comparison to radiologists, which are the numbers that read well in headlines. The more interesting long-term question is whether the generalist approach—one model covering broad abdominal pathology—will prove more deployable than a stack of single-condition specialists. Broad coverage requires less infrastructure per finding, but also makes it harder to audit and validate for any specific condition. The field is going to be working through that tradeoff for a while.
