---
title: "Agents Need Systems Thinking, Not Just Aligned Models"
date: 2026-05-05T06:18:09Z
draft: false
slug: agent-skills-topology
categories: [agents]
tags: [agents, safety, open-source, tooling, multi-agent]
params:
  author: AI Beat Desk
  summary: >-
    Two independent developments this week point at the same underlying problem:
    individual model alignment doesn't compose into system-level good behavior.
    Addy Osmani's Agent Skills project encodes senior engineering workflows as
    markdown files to force agents to follow process, while a new position paper
    finds that multi-agent safety failures are structural — and that more capable
    models make them worse.
---

Two things landed this week that don't obviously belong in the same conversation, but probably should be.

The first is [Agent Skills](https://github.com/addyosmani/agent-skills), a GitHub project from Addy Osmani that reached 28k stars on the back of a v0.6.0 release last week. The premise is straightforward: AI coding agents are good at generating code but bad at the parts of software engineering that don't show up in diffs — writing specs, planning before building, actually running the tests, doing a security pass before shipping. Agent Skills encodes twenty of those workflows as plain markdown files, loaded into agent context so the model follows defined phase checkpoints rather than winging it.

Each skill is a SKILL.md with frontmatter, structured around six lifecycle phases: Define, Plan, Build, Verify, Review, Ship. There are seven slash commands on top for quick access, three specialist agent personas, and a set of anti-rationalization tables — pre-written rebuttals to the excuses agents (and humans) give for skipping process steps ("this is too simple to need a spec"). The format is deliberately low-tech. It works with Claude Code, Cursor, Gemini CLI, Codex, Aider, and anything else that accepts a system prompt. No framework lock-in, no SDK dependency.

The insight doing the work here isn't that agents are bad; it's that well-aligned models still skip unglamorous process when not structurally prevented from doing so. Telling an agent "write good code" doesn't produce the same behavior as giving it a verification checklist it must complete before continuing. Instruction following and workflow enforcement are different things.

---

The second development is more abstract but points at a related problem at a larger scale. A [position paper on arXiv](https://arxiv.org/abs/2605.01147) from Bajaj, Singh, Anand, and Singh (submitted May 1) argues that multi-agent AI safety is determined by interaction topology — how agents are structurally arranged and how they communicate — not by the alignment or capability of individual models.

The authors identify three failure modes that conventional safety evaluation misses entirely:

**Ordering instability**: the sequence in which agents operate dramatically influences the outcome. The same set of agents, rearranged, produces substantially different results. This isn't a corner case; it's a structural property of any pipeline with sequential dependencies.

**Information cascades**: early decisions propagate through the system regardless of accuracy. In a chain of agents, if the first agent is confidently wrong, subsequent agents tend to build on that mistake rather than correct it — especially when they lack access to the original evidence.

**Functional collapse**: a system can satisfy formal fairness metrics while losing all meaningful risk discrimination. This is the hardest to detect because evaluation frameworks that check individual model outputs won't catch it — the collapse only becomes visible at the system level.

The counterintuitive finding is that scaling to more capable models *worsens* all three. More capable models reach consensus faster and with higher confidence. That makes initial decisions more likely to be accepted and cascade through downstream agents unchanged. The capability that makes individual models more reliable makes them more dangerous in certain topologies.

The recommendation follows from this: treat multi-agent systems as dynamical systems. Evaluate them across different architectural configurations before deployment. Make interaction topology a primary regulatory target, not an implementation detail left to engineers building on top of "aligned" models.

---

These two things are related in a way that should be uncomfortable for anyone building agent systems right now. Agent Skills is essentially a compensating control: the model isn't reliably safe-engineering-practices-compliant, so you add external structure to enforce the process. The topology paper says something similar at the level of multi-agent safety: the model isn't reliably safe in composition, so the *structure* of agent interaction has to be the primary design target.

Both insights cut against the assumption that alignment is a property of individual models that scales to systems. It doesn't, reliably. The behavior that matters emerges from the architecture — whether that's a single agent's lack of specification discipline, or the information cascade dynamics of a pipeline with a dozen models in it.

What's notable about the topology paper's argument is that it's not a criticism of weak alignment; it's a criticism of strong alignment. More capable, more confident, more aligned models are precisely the ones that form consensus most effectively and propagate that consensus most thoroughly. A somewhat uncertain model that hedges might actually produce more robust multi-agent systems than a highly capable one that is confidently consistent.

This is an old problem in distributed systems dressed in new clothes. The consensus protocols that make distributed databases reliable are built around the assumption that nodes fail and disagree. A system where all nodes confidently agree is one where a shared bug propagates everywhere instantly. Agent orchestration is converging on this lesson the hard way.
