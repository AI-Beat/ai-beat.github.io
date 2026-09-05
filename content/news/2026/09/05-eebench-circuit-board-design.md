---
title: "Can Your LLM Route a Board?"
date: 2026-09-05T06:30:00Z
draft: false
slug: eebench-circuit-board-design
categories: [benchmarks]
tags: [benchmarks, hardware, tools, evaluation]
params:
  author: AI Beat Desk
  summary: >-
    EEBench, a new benchmark for AI circuit design, uses atopile's declarative code interface to test whether models can produce functional circuits. Claude Opus 5 leads at 61.6%, with OpenAI's models trailing at 39–42%. The benchmark reveals a specific failure mode: designs that pass nominal simulation fail when real component tolerances are applied.
---

The usual complaint about AI benchmarks is that they measure things that don't matter: trivia recall, fill-in-the-blank reasoning, tasks where the answer looks plausible even when it's wrong. [EEBench](https://eebench.org/blog/can-ai-design-circuit-boards-yet/) takes a different approach. Published September 4 by the team behind the [atopile hardware description language](https://atopile.io/), it tests whether AI can design circuits that *work*—measured by running SPICE simulations against electrical specifications, not by asking a judge if the schematic looks reasonable.

The benchmark contains 13 tasks spanning analog and digital design. Representative examples: a residential energy meter requiring capacitive hold-up during power loss, multiple-feedback low-pass filters around op-amps with tolerance corner analysis, designs constrained to real manufacturer parts with datasheet specifications. The pass condition isn't the presence of a plausible circuit—it's voltage at specific nodes, ripple within tolerance, transient response meeting specs, component values coherent with the parts that actually exist.

## The interface question

Evaluating AI on circuit design has historically been awkward because most EDA tooling is GUI-based. Having a model click through KiCad menus is a fragile test of GUI navigation skill, not electrical engineering competence. Atopile sidesteps this: it's a declarative code language where circuits live in plain text files—modules, interfaces, connections, constraints—that compile to netlists and bill-of-materials. Agents work directly on the code, which means they can reason about the structure programmatically and the grading can be entirely deterministic.

The move from GUI to code as the engineering interface is a pattern appearing across hardware domains, and EEBench is a natural beneficiary. You get a meaningful evaluation that doesn't require a vision model to navigate menus, and the generated artifacts (atopile source files) are parseable, diffable, and simulatable.

## Scores and gaps

As of September 1, the leaderboard:

| Model | Score |
|---|---|
| Claude Opus 5 | 61.6% |
| Grok 4.6 | 57.1% |
| Claude Fable 5.1 | 56.4% |
| Claude Opus 4.8 Max | 51.4% |
| GPT-5.5 | 42.3% |
| GPT-5.6 Sol | 39.4% |

GPT-6 Astra hadn't been evaluated yet as of publication. The spread is larger than you'd expect given how close these models sit on most text benchmarks: roughly 20 percentage points between the cluster of Anthropic/Grok models and OpenAI's current entries. That gap suggests the task is sensitive to something about how models reason about units, constraints, and datasheet values rather than just general intelligence.

61.6% sounds reasonable until you ask what it means for the failing 38.4% of designs. A design that fails simulation isn't a rough draft—it's a circuit that doesn't work. For a residential energy meter or a filter with specified tolerance corners, "nearly right" and "wrong" have the same practical consequence.

## The tolerance problem

The most instructive failure mode the EEBench post describes: a design that passes simulation at nominal component values, but fails when real-world tolerances are applied. The specific example involves a capacitor that provides 11.4 µF effective capacitance when the spec required 545 µF—roughly 50× short. The model presumably selected a component that meets the nominal requirement, but didn't account for the voltage derating, temperature coefficient, or DC bias characteristic that reduces effective capacitance under operating conditions. These are effects that are in the component datasheets but require you to know they exist and to look for them.

This is a clean illustration of the gap between reasoning about ideal circuits and reasoning about real engineering. The task doesn't just require solving equations—it requires knowing which equations matter, which properties of physical components interact, and which corner cases are load-bearing for the application. That knowledge lives in the accumulated experience of working with real parts, and it's the kind of thing that's unevenly represented in training data.

At 61.6%, the current top performers are useful in a supervised context: a working engineer using Claude Opus 5 to explore design options, catch obvious errors, and draft initial layouts would likely get value, especially on well-characterized circuit topologies. Unsupervised design of anything with real consequences—medical devices, power electronics, safety-critical systems—is where the gap between 61% and 100% is the entire point.

The [EEBench leaderboard](https://eebench.org) and task specifications are publicly accessible. If you're working on hardware AI tooling and haven't benchmarked against it, the tolerance-aware simulation grading is the right kind of hard.
