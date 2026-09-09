---
title: "The Millennium Problem and the Credit Dispute"
date: 2026-09-09T16:05:14+0000
draft: false
slug: navier-stokes-openai-credit
categories: [research]
tags: [openai, mathematics, navier-stokes, millennium-prize, ethics, science]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI's agents proved that 3D Navier-Stokes equations can develop
    singularities — a genuine mathematical achievement. But the credit
    dispute that followed, involving an NYU mathematician and an Anthropic
    employee who'd been working the same problem for a year, raised a question
    that has no clean answer: what happens to independent research when an AI
    lab can decide to race at it the moment they hear a rumor?
---

On September 8, [OpenAI announced](https://openai.com/index/navier-stokes-solution/) that its autonomous agents had resolved one of mathematics' most famous open problems: the Navier-Stokes existence and smoothness problem, one of the seven [Millennium Prize Problems](https://en.wikipedia.org/wiki/Millennium_Prize_Problems) worth $1 million each. The answer they found is the one most mathematicians had suspected: smooth solutions to the 3D equations can [blow up in finite time](https://www.quantamagazine.org/ai-has-solved-one-of-maths-1-million-millennium-prize-problems-20260908/) — a singularity forms, where a point in the fluid theoretically moves at infinite velocity. Turbulence, it turns out, is even stranger than it looks.

That's a genuine result. The Navier-Stokes problem has been open for over a century, and the blowup question — whether smooth initial conditions can lead to a mathematical catastrophe — is physically meaningful, not just formally interesting. The proof builds on a framework developed by [Diego Córdoba](https://www.icmat.es/members/researchers/diego-cordoba-gazolaz/) (Institute for Mathematical Sciences, Madrid) and Luis Martínez-Zoroa (CUNEF University), whose technique of constructing "infinite cascades" of layered solutions provided the theoretical pathway. [Quanta Magazine's coverage](https://www.quantamagazine.org/ai-has-solved-one-of-maths-1-million-millennium-prize-problems-20260908/) is clear that the intellectual debt to that earlier work is significant.

What turned this into a controversy is the timeline — and what OpenAI did when they learned about it.

[Tristan Buckmaster](https://cims.nyu.edu/~buckmaster/), an NYU mathematician who has spent years on Navier-Stokes, had been collaborating with Levent Alpöge — a mathematician currently employed at Anthropic — for nearly a year. Working with publicly available models from both OpenAI and Anthropic, they produced a proof that a *simplified* version of the Navier-Stokes equations can break down. By August 15, they had their breakthrough. They were preparing to publish.

On September 1, OpenAI researchers heard rumors that two Millennium Prize problems had been solved. [According to MIT Technology Review](https://www.technologyreview.com/2026/09/08/1143747/what-openais-latest-controversy-tells-us-about-the-future-of-math/), OpenAI then launched an internal model — described as significantly more capable than GPT-6 Astra — at the remaining problems, running roughly 10,000 agents concurrently, spending millions of dollars over 88 hours. They proved the full equations can blow up, not just the simplified version. They published on September 8 — the same day Buckmaster posted his own work to Mastodon.

OpenAI then presented Buckmaster with two options, as [Axios reports](https://www.axios.com/2026/09/08/openai-math-solution-navier-stokes-credit): post independently, with OpenAI publishing a day later, or collaborate on a joint paper — but without Alpöge, because of his Anthropic affiliation.

The exclusion-for-competitive-reasons clause is where it gets genuinely uncomfortable. Alpöge had been part of this project for a year. That he works for a competitor apparently made him ineligible to be a named author on a joint paper. Buckmaster rejected both options and posted independently anyway. The joint paper did not happen.

[OpenAI's statement](https://www.axios.com/2026/09/08/openai-math-solution-navier-stokes-credit) is careful: "We (the researchers and the agents) did not see any of their work through any means until they released it publicly." But in the same breath they acknowledged they "cannot rule out that de-identified data derived from their usage of our products helped improve our models." That's a distinction that matters to lawyers and probably not much to Buckmaster. The researchers used OpenAI's tools extensively on a project that OpenAI then raced to complete after learning of its existence.

What exactly constitutes independent discovery in this environment is not a question with a clean answer. Both teams used Córdoba and Martínez-Zoroa's foundational framework. OpenAI proved the harder result. But the reason OpenAI knew to try in the first place was that someone was already a year into it. And the model they used may have been shaped, in some partial way, by the sessions Buckmaster and Alpöge ran on it.

OpenAI says it will not claim the $1 million Millennium Prize, framing the result instead as evidence of their systems' improving capabilities. That framing is accurate — the result does demonstrate something real about what large-scale agent deployment can do in a focused mathematical domain. The prize money was never the point.

The broader question it raises for science is harder. If an AI lab can spin up 10,000 agents at a problem the moment they hear a rumor it's close to solved, what does independent research mean when you're working on a hard open problem and using the lab's own tools to do it? The rumor network in mathematics is thin and fast. Problems that are nearly solved tend to be known to be nearly solved. The asymmetry between a researcher's individual capability and a lab's ability to redirect enormous compute toward a specific target isn't new — grants and corporate R&D have always introduced funding asymmetries — but the timescale is new. Eighty-eight hours, from rumor to proof.

Buckmaster is clear that he's not alleging misconduct in a legal sense. He's raising a question about norms that the field doesn't have a ready answer for.
