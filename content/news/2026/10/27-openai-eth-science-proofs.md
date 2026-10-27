---
title: "If the ETH Disproof Holds, It Rewrites Complexity Theory"
date: 2026-10-27T06:15:00Z
draft: false
slug: openai-eth-science-proofs
categories: [research]
tags: [research, formal-verification, complexity-theory, openai]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI announced that its models produced AI-generated manuscripts for three
    major scientific problems—including a disproof of the Exponential Time Hypothesis—all
    formalized in Lean and now under independent expert review. The claims are extraordinary,
    the formal proofs are real, and the gap between "Lean accepts this" and "the community
    accepts this" is where the interesting work begins.
---

On Sunday, OpenAI [announced](https://www.unite.ai/openai-says-its-models-produced-three-proofs-in-math-cs-and-physics/) that its internal models produced AI-generated manuscripts for three long-standing scientific problems: a proof of the local smoothing conjecture (which implies the Kakeya and Stein restriction conjectures in all dimensions), a disproof of the Exponential Time Hypothesis (ETH), and a rigorous existence proof for certain 4D quantum field theories including QCD—touching a Clay Millennium Prize problem. All three were formalized in Lean and are now under independent expert review organized by academic shepherds.

The ETH result is the one that should make every complexity theorist stop and reread the headline. ETH was formulated by Impagliazzo and Paturi in 1999 and has since become a foundational assumption underlying thousands of lower-bound proofs across parameterized complexity, approximation algorithms, and fine-grained complexity theory. It states, informally, that \(k\)-SAT cannot be solved in subexponential time. If OpenAI's claimed disproof holds, the implication is that for fixed \(k \geq 2\), a deterministic machine can exactly count the satisfying assignments of a \(k\)-CNF formula in subexponential time and polynomial space. That's not just refuting a conjecture—it would require re-examining the foundations of a large swath of theoretical CS.

The formalization matters and also doesn't settle the question alone. Lean checked the formal argument; what it cannot guarantee is that the formal statement captures precisely what ETH claims. The mismatch between a formal proof and its informal mathematical meaning is a real failure mode, and it's one reason OpenAI brought in Russell Impagliazzo, Mohan Paturi, and Ryan Williams to organize an open, independent examination. Williams confirmed he was asked to help. OpenAI has also [published the Lean proof](https://openai.com/index/building-scientific-understanding-with-academic-communities/) in its public GitHub repository and committed to making raw data public and funding a conference on the implications.

This is the right structure for an extraordinary claim. Lean formalization raises the bar above a natural-language preprint—someone has to actually find a gap in the formal argument, not just a hand-wavy step in a PDF. But expert scrutiny is still the only thing that converts "Lean accepts this" into "the mathematics community accepts this." The ETH formalization was dated October 17; the announcement came October 26. That's nine days from formalization to public release, which is fast by any standard.

The physics result is being handled differently: David Gross's group chose to write up the Yang–Mills existence result themselves rather than release OpenAI's artifacts directly. That restraint is interesting—it suggests the physics community has its own standards for what counts as a proof, independent of whether a proof assistant accepted it.

What makes this moment notable is not just the specific results, but the fact that formal verification at this scale is now part of the workflow. It used to be that an AI "discovering" a theorem meant producing a natural-language sketch that a human then had to verify manually. The combination of a capable reasoning model and a proof assistant like Lean changes what it means to check a claim. The check is still not automatic—you have to verify the formalization faithfully captures the original problem—but it's a qualitatively different kind of evidence than a PDF on arXiv.

The prudent position right now is: these are remarkable claims backed by formal artifacts, under serious independent review, from credible mathematicians and physicists. That's the most favorable condition any extraordinary claim could be in. Whether they survive scrutiny, and what it means if they do, is the story to watch over the next few months.
