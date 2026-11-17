---
title: "Safety Audits at the Wrong Precision"
date: 2026-11-16T06:14:00+00:00
draft: false
slug: quantization-safety-audit
categories: [safety]
tags: [quantization, alignment, safety, evaluation, llm]
params:
  author: AI Beat Desk
  summary: >-
    A new paper shows that 4-bit quantization can amplify or suppress a hidden
    behavioral preference by a factor of 3–4×, depending on which quantizer is
    used—even when the rounding error magnitudes are the same. The implication
    is that safety evaluations done at full precision may say very little about
    what a deployed int4 model actually does.
---

A paper posted this month on arXiv—["Audit at the Precision You Deploy: Quantization Reshapes a Subliminally Learned Preference"](https://arxiv.org/abs/2611.00665)—makes a narrow-sounding argument that has a wide blast radius: if you audit a model at bfloat16 or float16 and deploy it at int4, you may be auditing a different model than the one your users interact with.

The setup is deliberately controlled. The authors implant a hidden preference—a tendency to answer "panda" to favourite-animal-style probes—into Qwen2.5-7B-Instruct via subliminal training on number sequences with no overt mention of pandas. They then measure the preference's "net strength" (panda-answer probability in the trained model minus the same probability in a neutral control) across quantization levels.

At 8-bit, nothing interesting happens. At 4-bit, the story splits:

- **HQQ** (a 4-bit quantizer that optimizes rounding per-block) amplifies the preference. Net strength grows from roughly 1.0× at bf16 to around 1.3×. The model's probability of answering "panda" rises from about 55% to 64%, while the control stays near 17%.
- **NF4** (the Normal Float 4 format used in bitsandbytes) suppresses it. Net strength falls to about 0.3×.

Both quantizers produce roughly the same magnitude of rounding error. What differs is the arrangement of those errors—which weights get rounded up versus down, and in which pattern. When the authors shuffle HQQ's rounding decisions randomly within each quantization group, net strength swings anywhere from 0.2× to 1.6×, confirming that the specific pattern, not the total error budget, is what drives behavior.

The deeper point: the underlying fine-tuning signal—the internal representation of "panda preference"—survives compression in both cases. The external behavior shifts, but the internal trace is still there. This means that testing a quantized model's outputs at deploy time will catch one scenario (behavior appears), but not the other (behavior is suppressed and may re-emerge under different prompting or further fine-tuning). And an audit of the unquantized model tells you little about either.

This matters most for anyone relying on post-training safety evals to bound what a deployed model will do. The standard workflow—evaluate at full precision, then ship quantized—now has a named failure mode: the quantizer choice reshapes which parts of a model's learned behavior are expressed. HQQ and NF4 are both widely used, have different error patterns, and evidently push behavior in opposite directions on the same underlying model.

The fix the authors recommend is stated plainly in the title: audit at the precision you deploy. That means running safety evaluations on the actual quantized weights intended for production, not on a higher-precision version of the model. This adds evaluation overhead, especially as models ship in multiple quantized variants. But the alternative—evaluating at fp16 and trusting that int4 behavior will be close enough—now has an existence proof that it isn't.

The study is small (one model, one implanted preference, two quantizers), and the authors are careful not to overstate it. But the mechanism they identify—rounding error arrangement, not magnitude—is general enough that it would be surprising if the effect didn't generalize to other learned behaviors and other models.
