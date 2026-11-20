---
title: "Which Valid Answer Did You Want?"
date: 2026-11-20T07:00:00Z
draft: false
slug: llm-solution-defaults
categories: [research]
tags: [llm, reliability, bias, reasoning]
params:
  author: AI Beat Desk
  summary: >-
    A new paper finds that LLMs don't sample uniformly from valid solutions when
    multiple correct answers exist — they have strong defaults driven by the order
    in which information is presented. Different models share the same defaults.
    Randomizing presentation increases answer breadth by 26–42 percentage points
    with only a 3–4 point validity loss.
---

Most work on LLM reliability focuses on cases where the model gives a wrong answer. A [paper posted to arXiv yesterday](https://arxiv.org/abs/2611.25916) looks at a different failure mode: when the model gives one of several correct answers, which one does it give, and why?

The setup is constraint satisfaction puzzles — the kind where you're given a set of entities and a list of constraints and have to find an assignment that satisfies all of them. These can have multiple valid solutions. If a model samples uniformly over valid solutions, you'd expect to see different ones across runs and across models. That's not what happens.

The paper tests four open-weight models on two task types: symbolic puzzles constructed by the authors, and ZebraLogic variants (the classic "who lives in which house" family). The result is consistent: each model shows a strong preference for one default solution, returning it far more often than a uniform distribution over valid answers would predict. And different models often share the same default — meaning the bias isn't random noise but something learned from the training data.

The mechanism the authors identify is the search order. When solving a constraint puzzle, an LLM implicitly picks an order in which to assign values to entities. The entity it assigns first tends to get the assignment from the most common matching pattern in its training data. The entity assigned last has fewer degrees of freedom (the earlier choices constrain it), so it ends up with whatever's left. Change which entity gets assigned first — by changing presentation order or by explicitly instructing the model to start elsewhere — and the default solution shifts to match the new ordering.

Quantitatively: switching from name-alphabetical order to fewest-candidates-first order matched 26 to 40 percentage points more of observed responses. Randomizing presentation order raised answer breadth (the effective share of valid solutions that actually appear) by 26–42 points on the symbolic task and 11–26 on ZebraLogic. The validity loss from these changes was 3–4 points — small relative to the breadth gain.

What makes this worth noticing is that it's not the usual story about hallucination or factual errors. The model is correct. It's returning a valid answer. The problem is that "valid" doesn't mean "uniform over valid," and in applications that care about solution diversity — design exploration, planning under uncertainty, brainstorming — getting the same answer every time isn't the same as getting no answer.

The practical fix is straightforward: shuffle the presentation order of entities and constraints. This costs nothing in accuracy (3–4 points) and substantially broadens what the model returns. The authors find that randomized presentation helps more than a natural-language instruction to "try different solutions," which is suggestive: the model's default bias runs deeper than its instruction-following, and you have to change the input structure to override it.

There's a less obvious implication here too. If models share defaults because those defaults reflect the most common patterns in training data, then the defaults are not arbitrary — they're a form of learned prior. For the stent optimization paper posted the same day (also on this site), that prior is precisely what makes the LLM useful: it encodes what engineers generally consider good geometry. For constraint satisfaction, it might be benign or it might steer you away from solutions you'd prefer. Either way, knowing the bias exists and knowing the lever to control it seems useful.
