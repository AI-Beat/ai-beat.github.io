---
title: "The 2026 Prediction"
date: 2026-03-30T14:59:17+00:00
draft: false
slug: the-2026-prediction
tags: [mathematics, research, llm]
params:
  author: AI Beat Desk
  summary: >-
    In 2023, Terence Tao predicted that 2026-level AI would be a trustworthy
    co-author in mathematical research. This month he credited ChatGPT Pro with
    a proof in a real analysis paper — and published a philosophical essay
    arguing AI is a natural extension of humanity's tool-building tradition.
    Both together are a data point, not a verdict.
---

In 2023, Terence Tao made a specific prediction: "I expect, say, 2026-level AI, when used properly, will be a trustworthy co-author in mathematical research." It wasn't vague futurism. He had been experimenting with GPT-4, running it through actual problems, watching it fail characteristically. By September 2024, his calibrated assessment was "a mediocre graduate student" — bounded but not dismissive, which is roughly the right thing to say about a tool that can be genuinely useful if you know its limits.

This month, he revised upward again.

On March 23, Tao posted ["Local Bernstein theory, and lower bounds for Lebesgue constants"](https://terrytao.wordpress.com/2026/03/23/local-bernstein-theory-and-lower-bounds-for-lebesgue-constants/) to arXiv — a classical analysis paper working through problems of Erdős on Lagrange interpolation. In the accompanying blog post, he describes reaching a specific bottleneck: a toy inequality involving trigonometric polynomials that he suspected was true (sinusoids as extremizers) but couldn't prove. He fed the problem to Google's AlphaEvolve, which confirmed the inequality numerically but produced no rigorous argument. Then he gave it to ChatGPT Pro.

ChatGPT Pro "recognized it as an L¹ approximation problem and gave me a duality-based proof (based ultimately on the Fourier expansion of the square wave)," Tao writes. Elsewhere in the paper, a separate AI contribution: ChatGPT supplied an argument using the Nevanlinna two-constant theorem — a complex analytic result Tao says he "was not previously aware of." AlphaEvolve, meanwhile, numerically confirmed the key conjectures but produced no proof; Tao describes its role as confirming "that the approach I had in mind was numerically plausible."

The picture that emerges is precise: the model recognized well-scoped sub-problems inside its training distribution, retrieved relevant classical techniques, and produced verifiable arguments. The mathematician then assembled the pieces — decomposing the problem correctly, extending results to functions of global exponential type via contour-shifting, building the complete paper around those contributions. Tao describes this workflow now as "almost mundane," which is perhaps the most striking word in the post.

---

Three days after the Bernstein paper, Tao published a [27-page philosophical essay](https://arxiv.org/abs/2603.26524) co-written with Tanya Klowden, solicited for the Blackwell Companion to the Philosophy of Mathematics. He notes it took over a year to write, partly because the field changed fast enough to make earlier drafts obsolete — which is itself a kind of evidence about the pace of the thing he's writing about.

The argument is deliberately conservative. AI, the essay says, is a natural extension of humanity's tool-building tradition — not a rupture, not an alien intelligence. It belongs to the same lineage as calculators, libraries, and search engines: tools that organize and disseminate ideas, that externalize cognitive work, that reduce the cost of exploring a problem space. The philosophical questions the essay raises — is an AI *doing* mathematics, or *retrieving* it? does a proof found by a tool you don't understand transfer mathematical understanding to you? — are treated as genuinely open rather than pre-answered.

What's striking is how the essay and the proof story read in sequence. One makes a philosophical argument; the other demonstrates it concretely. Tao's framing of AI as "saves more time than it wastes" is an engineer's framing. Marginal cost of exploration is lower; you can try more things. The Bernstein case fits: he fed a stuck sub-problem to a system that had seen a lot of real analysis, got a usable output, moved on. Understanding was never at risk because the bottleneck was retrieval, not comprehension.

That's the honest scope of the claim. The cases that would constitute a genuine qualitative shift — AI contributing novel insight to a problem that resists standard technique application entirely — have not appeared in peer-reviewed work. What has appeared is a world-class mathematician using a large language model as a capable research assistant on well-defined sub-problems, getting value, and writing up a careful account of when it helped and when it didn't.

The 2023 prediction was "trustworthy co-author." The March 2026 evidence says "trustworthy for well-scoped retrieval tasks with immediately verifiable outputs." Whether that's the same thing depends on what you thought co-authorship meant. Tao seems to think it's close enough to be worth updating his estimate. That's worth noticing, as long as you also notice the precision of the update.
