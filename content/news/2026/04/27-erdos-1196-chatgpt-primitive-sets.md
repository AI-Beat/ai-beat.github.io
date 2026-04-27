---
title: "The Wrong First Move"
date: 2026-04-27T06:39:35Z
draft: false
slug: erdos-1196-chatgpt-primitive-sets
categories: [ai-research]
tags: [mathematics, llm, reasoning, openai, gpt]
params:
  author: AI Beat Desk
  summary: >-
    GPT-5.4 Pro solved Erdős Problem #1196 — a 1968 conjecture about primitive
    sets — when a 23-year-old amateur fed it the problem in a single prompt. The
    AI's approach used von Mangoldt weights and a downward Markov chain, a
    framing that existed in analytic number theory for ninety years but had never
    been applied here. Terence Tao's explanation for why experts missed it is the
    most telling part of the story.
---

On April 13, Liam Price — 23 years old, no graduate training in mathematics — typed [Erdős Problem #1196](https://www.erdosproblems.com/latex/1196) into GPT-5.4 Pro. About eighty minutes later, the model had produced a proof sketch. Price posted it to the [Erdős Problems discussion forum](https://www.erdosproblems.com/forum/thread/1196). Kevin Barreto, a second-year undergraduate at Cambridge, recognized that something unusual had happened and passed it on. Terence Tao and Jared Lichtman are now working on a cleaned-up version.

The problem itself is easy to state. A *primitive set* is a collection of positive integers in which no element divides any other. Primes form one; so does {6, 10, 15}. Erdős, Sárközy, and Szemerédi conjectured in 1968 that for any primitive set \(A \subseteq [x, \infty)\), the sum

\[\sum_{a \in A} \frac{1}{a \log a}\]

approaches exactly 1 as \(x \to \infty\). The conjecture had remained open for 58 years. It is not that people ignored it — it is that attempts to approach it via the natural tools of combinatorial number theory repeatedly stalled.

What GPT-5.4 Pro found was a reframing. Rather than working directly with the divisibility structure, the model reached for the [von Mangoldt function](https://en.wikipedia.org/wiki/Von_Mangoldt_function) — an object from analytic number theory that encodes the prime factorization of integers — and constructed a downward Markov chain: a random walk on the integers defined by the transition \(n \mapsto n/q\) with probability proportional to \(\Lambda(q)/\log n\), where \(\Lambda\) is the von Mangoldt function. The key observation is that under this Markov process, primitive sets have a natural invariant measure, and the conjectured sum falls out of the stationarity condition.

Markov chains and von Mangoldt weights are both standard tools. Neither is exotic. What is notable is that nobody had pointed them at each other in this context. According to Tao, the model's approach reveals "a previously undescribed connection between the anatomy of integers and Markov process theory" — a connection with potential consequences beyond this particular conjecture.

The most striking part of Tao's public commentary is not the praise. It is the diagnosis. He said that experts collectively made "a slight wrong turn at move one." Not that the problem was uniquely hard or that the tools were missing. Just that human mathematicians, approaching the problem with existing frameworks and intuitions, consistently started from premises that made the right path invisible. The model, apparently without those priors, tried something that competent experts would have ruled out early.

There is a recurring pattern in AI-assisted mathematical results: the AI output is described as "rough" or "quite poor" as raw text, requiring expert interpretation to extract the actual mathematical content. This case fits that pattern exactly. Price's initial post was a cleaned-up version of model output; Barreto recognized that it contained something real; Tao and Lichtman are now doing the work of turning the insight into a proof with correct notation, verifiable steps, and appropriate citations. The model contributed the key idea. The humans are verifying and packaging it.

That division of labor matters. It is not "AI solved a math problem" and it is not "AI produced a hallucinated proof that looked plausible." It is something more specific: a large language model, queried by someone with no domain expertise, surfaced a mathematical perspective that domain experts had not tried, and that perspective turned out to be the right one. The model did not know why it was right. It did not know that it contradicted decades of expert approaches. It just generated text that, when interpreted by people who understood the math, contained a valid path.

Tao's comment about the "wrong first move" is worth sitting with. If true, it suggests that some open problems have been stuck not because the proof is hard but because the field developed a consensus on which approaches are worth trying, and that consensus was wrong. A system without that consensus — or with a different one — might find the path directly. This is a different mechanism than "AI is better at math than humans." It is closer to: AI occasionally tries things experts have learned not to try, and sometimes experts learned wrong.

The full proof, once Tao and Lichtman have finished it, will be worth reading for the mathematical content. What it will not tell you is how the model got there. That part may remain opaque.
