---
title: "The Homework CLAUDE.md"
date: 2026-06-02T06:14:00+00:00
draft: false
slug: cs336-claude-md-agent-constraints
categories: [agents]
tags: [agents, education, claude-code, coding-agents, project-files]
params:
  author: AI Beat Desk
  summary: >-
    Stanford CS336 shipped a CLAUDE.md file in its assignment repositories that
    instructs coding agents to act as Socratic tutors rather than solution
    generators. It is a small thing technically and a significant thing
    conceptually: domain-specific behavior specification embedded directly in
    the project.
---

Stanford's [CS336: Language Modeling from Scratch](https://cs336.stanford.edu/) is a notoriously difficult course — five heavy implementation assignments covering tokenization, FlashAttention, scaling laws, Common Crawl data processing, and alignment via reinforcement learning, all with minimal scaffolding. The philosophy is that you learn to build language models by actually building one, not by reading papers about it.

The course has an AI usage policy that permits asking LLMs conceptual questions but prohibits using them to directly solve problems. That is a common policy. What is less common is that the course staff also published a [`CLAUDE.md`](https://github.com/stanford-cs336/assignment1-basics/blob/main/CLAUDE.md) file in the assignment repositories — a project-level configuration file that Claude Code (and other tools that read `CLAUDE.md`) loads automatically when opened in a project directory.

The file instructs the agent with unusual specificity. It should not write Python or pseudocode. It should not complete `TODO` blocks. It should not point to third-party implementations. When a student asks how to implement a tokenizer, the agent should ask what they have already tried, reference the relevant lecture materials, and guide them toward the answer — but not produce the answer. The framing in the file is direct: agents should not give students "the solution or idea for how to solve a problem."

What makes this technically interesting is less the policy itself — that is just pedagogy — and more the mechanism. Claude Code's project file system was designed primarily as a way to tell agents about codebases: style conventions, build systems, which files to ignore. Using it to encode domain-specific behavioral constraints is a natural extension, but CS336 seems to be one of the first courses to do it explicitly and publish the result.

The constraints are quite precise. The list of forbidden topics matches the course assignments almost perfectly: no help with tokenizer implementation, no transformer block code, no optimizer logic, no training loops, no Triton kernels, no distributed training. That specificity is the point — a general "be helpful but educational" instruction is easy to satisfy superficially, while an enumerated list of implementation areas makes the boundary clearer to the agent.

The [Hacker News thread](https://news.ycombinator.com/item?id=48359232) raised the enforceability question immediately: students can edit the file, delete it, or just use a different tool. This is true. The CLAUDE.md approach relies on the agent following the instruction honestly and on students not actively circumventing it — neither of which is guaranteed. One commenter noted that even if every constraint is bypassed, the file still signals to students what the course values, which has some pedagogical weight even absent technical enforcement.

The deeper question is whether CLAUDE.md-style constraints can hold up in practice when the agent is used at scale. An instruction set that fits in a few hundred tokens is easy to follow in a short context; in a long conversation where other instructions accumulate, the constraints may degrade. This is the same failure mode documented in the [constraint decay paper from May 25](/news/2026/05/constraint-decay-coding-agents/) — behavioral boundaries erode as context grows.

Still, as a pattern, it is worth paying attention to. What CS336 is doing is separating the question of what an agent should do in general from what it should do in a specific project context. That separation — capability versus configured behavior — is going to become more important as AI agents are deployed in more specialized domains with more specific requirements. A code review agent in a safety-critical codebase, a tutoring agent for a particular curriculum, a compliance agent for a regulated industry: all of them benefit from domain-specific behavioral specifications that live in the repository alongside the code they apply to.

The course itself runs through June 10, so the current iteration is nearly finished. Whether the CLAUDE.md approach helped student outcomes is not yet measured. What has already happened is that the file exists, is public, and will likely be read by educators designing similar constraints for other courses and contexts.
