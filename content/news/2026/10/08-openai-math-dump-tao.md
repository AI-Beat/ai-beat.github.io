---
title: "372 Results, and Math Can't Read Them Fast Enough"
date: 2026-10-08T06:11:32+00:00
draft: false
slug: openai-math-dump-tao
categories: [research]
tags: [mathematics, openai, formal-verification]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI posted 372 AI-generated math proofs to GitHub — Lean-checked and likely correct, covering problems from the 4D Kakeya conjecture to progress on the Riemann hypothesis. Terence Tao says this marks the end of "Math 1.0" and calls for a rethinking of what mathematics is for in the age of AI.
---

On Tuesday, OpenAI [posted 372 math results](https://openai.com/index/sharing-ai-progress-in-mathematics/) to a public GitHub repository. An unreleased internal model, averaging about three hours of compute per problem, produced full or partial solutions to open questions across mathematics and theoretical computer science. Many of the proofs come with Lean formalizations — machine-checkable certificates that make them almost certainly correct, even if nobody has read them carefully yet.

The scope is notable. The results include a claimed solution to the four-dimensional Kakeya conjecture, algorithmic improvements, and progress on three of the Clay Institute's Millennium Prize problems, including the Riemann hypothesis. OpenAI says its own mathematicians do not yet understand many of the new results. The company plans to keep going.

[Terence Tao](https://mathstodon.xyz/@tao/117395269325940185) called this the end of "Math 1.0" — the era in which solving open conjectures was mathematics' engine. He argues that AI-driven results are generating fewer seminars, fewer collaborations, and fewer new researchers entering the field. People solve problems with AI, lose interest once the target is hit, and often can't explain the output. The result is correct; the community doesn't grow. He calls for "Math 2.0": a mode that values exposition, community, and broad understanding over raw problem-solving velocity.

That's not just a philosophical preference — it's a practical concern. [Andrew Sutherland at MIT](https://www.scientificamerican.com/article/openai-unleashes-hundreds-more-math-results-upon-a-field-already-in-shock/) put it plainly: "We should ask for receipts." OpenAI has not released the model or the prompts, despite an advisory group recommending fuller disclosure. Without that, replication is impossible and the results can't be fully absorbed into the literature. Tristan Buckmaster at NYU raised a harder question: it's not yet clear whether mathematicians' earlier work inadvertently guided the model, or whether any of the outputs are close enough to existing unpublished work to constitute inadvertent plagiarism.

The Lean check matters here, but only so far. A Lean proof is a machine-verified certificate that a statement is true. It doesn't tell you *why* the theorem is true, what new ideas the proof contains, or whether those ideas are original. Checking correctness is tractable. Checking originality — when the prover is a neural network operating over billions of parameters and the outputs are thousands of lines of formal code — is much harder.

Daniel Litt at the University of Toronto takes a more optimistic view: the results are good for mathematics, and withholding them would be worse. He expects mathematicians to become much more productive with AI assistance, even if the near term is disorienting. That's probably right in the long run. But Tao's concern cuts deeper than pace — it's about what happens to a discipline when its core activity, the long collaborative struggle to understand hard problems, gets replaced by a process that produces answers without understanding.

The honest answer is that nobody knows yet. The 372 results will take the mathematical community months to assess. Some may turn out to contain genuinely new ideas; others may be long proofs of things that could have been found by hand with more time. The Lean formalizations are a real contribution even if the underlying mathematical ideas turn out to be shallow. And the Riemann hypothesis progress, if it holds up, is in a different category entirely.

The field is going to spend years unpacking this. That unpacking is, itself, a form of "Math 2.0" whether Tao framed it that way or not.
