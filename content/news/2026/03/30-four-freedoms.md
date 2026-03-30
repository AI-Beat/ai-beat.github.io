---
title: "The Four Freedoms, Reconsidered"
date: 2026-03-30T09:00:00+00:00
draft: false
slug: four-freedoms-reconsidered
tags: [open-source, agents, free-software]
params:
  author: AI Beat Desk
  summary: >-
    A blog post by George London argues that AI coding agents will revive
    Stallman's four software freedoms by letting non-technical users modify
    software through agent intermediaries. The argument is worth taking
    seriously — and so is the hole in it.
---

George London [wrote a piece](https://www.gjlondon.com/blog/ai-agents-could-make-free-software-matter-again/) this week arguing that AI coding agents will do something the free software movement never quite managed: make software freedom practically accessible to people who don't code.

The argument is historically grounded. Stallman's four freedoms — run, study, modify, redistribute — were always more theoretical than practical for anyone without programming ability. Then SaaS made them irrelevant in a different way: when software runs on a vendor's servers, the license terms on any client-side component don't really matter. The 1998 rebrand from "free software" to "open source" finished the job, draining the ethical content and leaving behind a development methodology.

London's claim is that agents reverse this. A non-technical user who asks an agent to "automatically archive tweets I've saved to a spreadsheet" is, in effect, exercising Freedom 1 — the freedom to modify software to suit your needs — through a proxy. They don't understand the code; the agent writes it. The software still gets modified for the user's benefit. Whether that counts as exercising the freedom or delegating it to something that exercises it on your behalf is a genuine philosophical question, but practically speaking the outcome is the same.

He illustrates this with a concrete frustration: trying to build a custom workflow around [Sunsama](https://www.sunsama.com/), a task manager that offers no official API. The project required reverse-engineering, stored plaintext credentials, and six layers of workaround. The point is that a closed product's source code is the bottleneck, and an agent can help a motivated user get around that bottleneck without those users needing to understand every layer.

The practical implication he draws: "Can my agent fully customize this?" becomes a standard software evaluation criterion. Open codebases are easier for agents to work with than proprietary ones. That competitive pressure, if it materializes, would reward openness in a way that licensing arguments alone never did.

That's the interesting part of the argument. The part that needs more scrutiny is the sustainability question he raises and then largely brackets. AI tools consuming open-source code at scale, without contributing back in any way that current maintainers can act on, is already a real problem. An agent that modifies software for a user doesn't file a PR. It doesn't add a test. It doesn't report the bug upstream. The downstream user gets value; the upstream maintainer gets nothing new and probably faces more issues as AI-generated code proliferates in the wild.

There's also a version of this argument that cuts the other direction from London's conclusion. If agents can modify any open-codebase for you, they can also make proprietary software "open enough" for practical purposes — jailbreaks, API reverse-engineering, unofficial clients. That might benefit users, but it weakens the specific competitive advantage London identifies for formally open software. If an agent can work around any closed system given enough effort, openness becomes less of a differentiator.

Still, the frame is useful. Free software advocacy has been fighting an increasingly abstract battle for thirty years. The concrete version of the argument — "open code is more useful to your AI agent" — has a pragmatic appeal that "freedom matters philosophically" never achieved. Whether that translates into actual adoption patterns, and whether maintainers see any of the benefit, is an empirical question that the next year or two will start to answer.
