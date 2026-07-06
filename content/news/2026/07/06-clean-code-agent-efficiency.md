---
title: "Clean Code Makes Cheaper Agents"
date: 2026-07-06T06:08:49+00:00
draft: false
slug: clean-code-agent-efficiency
categories: [agents]
tags: [agents, coding-agents, research, software-engineering]
params:
  author: AI Beat Desk
  summary: >-
    Two independent papers — a SonarSource study across 660 Claude Code trials
    and an ISSTA 2026 paper on structural annotations — converge on the same
    finding: the shape of a codebase changes how coding agents behave, not just
    how fast humans can read it. Clean code cuts agent token costs 7–8% and
    reduces file revisitations by 34%; explicit structural anchors halve
    run-to-run variance and improve localization. The environment is part of
    the model.
---

Two research groups, working independently on different questions, landed on variations of the same answer this summer: a coding agent's behaviour is shaped by the codebase it operates in, not only by the model driving it.

The first paper comes from SonarSource. Priyansh Trivedi and Olivier Schmitt built a ["minimal-pair" benchmark](https://arxiv.org/abs/2605.20049) — six repository pairs that are architecturally identical and behaviourally equivalent, but differ along code quality axes. They used two pipelines to generate the pairs: *Slopify*, which introduces static-analysis violations into clean code, and *Vibeclean*, which resolves violations in messy code. Thirty-three tasks, 660 trials through Claude Code, and the headline result is tightly split: code cleanliness has essentially no effect on task success (−0.9pp pass rate). But it has a meaningful effect on operational cost. Agents working on cleaner repositories used 7.1% fewer input tokens, 8.5% fewer output tokens, 11.1% fewer reasoning characters, and opened previously-edited files 34% less often.

The interpretation is worth sitting with. The model solved the problem either way. But on messier code it thrashed — backtracking to files it had already touched, burning tokens on navigation rather than edits. Code that a human would call "hard to read" was also code the agent found expensive to traverse.

The second paper, [accepted at ISSTA 2026](https://arxiv.org/abs/2606.26979), comes at the problem from the opposite direction: instead of removing noise, what happens when you add signal? The authors injected "CodeAnchor tags" — lightweight structural comments encoding call graphs, inheritance hierarchies, imports, and containment — directly into source files. Think `# invoked by: FooController.handle` and `# calls: BarService.fetch` as structured comments at the top of each function, generated statically and requiring no runtime infrastructure.

Across 274 SWE-bench Lite tasks with OpenAI Codex, Anchor-Topo (the basic call-graph + inheritance variant) improved function-level localization from 83.21% to 85.40% — a 2.2pp gain — and reduced the average number of tool calls per task from 35.3 to 33.7. More striking is what happened to consistency: run-to-run standard deviation roughly halved (from 0.043 to 0.022 on Lite), and single-run pass@1 rose 3.4pp. The agent wasn't meaningfully smarter, but it was more disciplined. It followed links rather than searching from scratch, took fewer wrong turns, and produced the same answer more reliably across repetitions. The cost was about 10% more input tokens per task — a reasonable trade for the reliability gain.

Together, the two papers sketch complementary interventions. Cleanliness is subtractive: reducing violation density removes navigational drag. Structural annotations are additive: injecting explicit graph relationships turns the codebase into something closer to a map. One makes the territory less confusing; the other adds signposts.

The practical implication is less obvious than it sounds. Neither paper argues that you should rewrite your codebase before pointing an agent at it. But they do suggest that the usual arguments for code quality — readability, maintainability, onboarding — now have a computational economics component. Technical debt isn't just a tax on future humans; it's a tax on agent token budgets, paid on every invocation. A 7% token reduction compounds over thousands of agentic tasks in a way that flat refactoring metrics don't capture.

What the structural annotation paper adds is a direction that doesn't require cleaning up anything: if your codebase already carries call graph information in a tool or language server, exposing it as inline text (or having a pre-processing step that injects it) could improve both agent reliability and, via reduced variance, the economics of running multiple samples. That's a pipeline change, not a codebase change.

The deeper point in both papers is that "model quality" is a partial picture. The environment the model operates in — its context — is doing work too. These are not surprising findings in principle; practitioners have suspected this for a while. But having controlled studies with precise numbers makes the tradeoffs legible in a way that intuition doesn't.
