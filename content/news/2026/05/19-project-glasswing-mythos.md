---
title: "When the AI Builds the Proof of Concept"
date: 2026-05-19T06:11:00+00:00
draft: false
slug: project-glasswing-mythos-cloudflare
categories: [security]
tags: [security, anthropic, vulnerability-research, ai-agents, offensive-security]
params:
  author: AI Beat Desk
  summary: >-
    Cloudflare tested Anthropic's Mythos Preview — a security-focused model
    released under Project Glasswing — against fifty of its own internal
    repositories. The model can do something earlier tools couldn't: chain small
    vulnerability primitives into working exploits, then write and run proof-of-
    concept code to confirm exploitability. Cloudflare's eight-stage agent
    pipeline is a detailed blueprint for how production-grade AI security research
    actually has to be structured.
---

The standard story about AI and vulnerability research goes like this: the model finds a candidate bug, flags it with appropriate caveats ("this might be exploitable"), and a human security researcher does the actual work of turning a theoretical flaw into a confirmed finding. The model handles volume; the human handles depth.

[Cloudflare's account of testing Mythos Preview](https://blog.cloudflare.com/cyber-frontier-models/) — Anthropic's security-focused model, released under [Project Glasswing](https://www.anthropic.com/glasswing) — describes something that disrupts that clean division. Mythos can reason about how to combine multiple small vulnerability primitives into a working attack chain, then write the triggering code, compile it in a sandbox, iterate when the first attempt fails, and produce a confirmed proof of concept. That last step — the transition from "suspected flaw" to "demonstrated exploitable" — is what has historically required a senior human researcher.

Cloudflare pointed Mythos Preview at more than fifty of its own repositories: the runtime, edge data path, protocol stack, control plane, and open-source projects. The goal was adversarial red-teaming of live internal code to find real problems before attackers do.

## What Actually Works

The strongest finding from Cloudflare's testing isn't that Mythos found bugs — it's that the findings Mythos generated didn't just stop at identification. The model demonstrated "reasoning comparable to senior security researchers" in assembling primitives: taking a memory disclosure here and an out-of-bounds write there and working through whether they could be composed into something that achieves privilege escalation or remote code execution. Generic code scanners have never done this. They find patterns. Mythos constructs arguments.

The proof generation capability matters because it changes the triage problem. A finding that comes with working exploit code, compiled and run in a sandbox, has a very different evidentiary weight than a finding that comes with a paragraph of hedged speculation. The latter requires a researcher to validate from scratch; the former is already validated.

## What Doesn't Work Yet

Cloudflare is honest that the noise problem is real. Two factors dominate false positives: language choice and model hedging. C and C++ codebases with direct memory access generate far more candidate findings than memory-safe languages — not because the analysis is wrong but because there are genuinely more classes of potential issues to reason about. And Mythos, like most capable models, qualifies its findings heavily ("possibly exploitable," "potentially affected"), which makes triage difficult when you're processing hundreds of candidates.

The safety guardrail behavior was also inconsistent in a way that matters for production use. On the same code, Mythos would sometimes refuse to perform vulnerability research, then agree after an unrelated change to the environment. The refusal wasn't triggered by the content being requested but by contextual factors that had nothing to do with the sensitivity of the task. That's a meaningful reliability problem for any team trying to automate security workflows — you can't build a pipeline on a component that sometimes says no for opaque reasons.

## The Pipeline That Made It Work

What's arguably more interesting than Mythos itself is the harness Cloudflare built to use it. A generic "give the agent a shell and some repos" approach doesn't work at this scale. Cloudflare describes eight distinct stages:

**Recon** maps the codebase architecture and trust boundaries before any vulnerability search begins. **Hunt** runs roughly fifty concurrent agents targeting specific attack classes in parallel. **Validate** runs independent verification with different prompts to filter candidates — exploiting the fact that a true vulnerability will show up consistently, while a false positive produced by one prompt tends to disappear under a differently framed query. Then **Gapfill**, **Dedupe**, **Trace**, **Feedback**, and **Report** progressively refine and consolidate findings.

This is what production offensive security with AI actually looks like, and it's more engineering than most people imagining "use AI to find bugs" have in mind. The concurrent agent architecture, the independent validation step, the deduplication — these are operationally important. Without them, what you get isn't security research; it's a large list of hedged guesses.

## The Broader Context

Project Glasswing is Anthropic's controlled research program for deploying Mythos: organizations test it against their own systems, findings are disclosed responsibly, and Anthropic learns how the model behaves on real production code in real adversarial contexts. Cloudflare is a natural first partner — a company that runs infrastructure at scale, has a sophisticated security team, and has the engineering capacity to build the kind of harness that makes the work actually useful.

The question this raises for the rest of the industry is less about whether AI can replace security researchers (it can't, yet) and more about where the floor is rising. If Mythos-quality capability becomes widely available — which it will, eventually — then the set of people who can run credible exploit development against a codebase expands considerably. The defense has to assume a higher baseline attacker capability. Cloudflare's detailed write-up reads partly as an advertisement for what their own security posture can handle, and partly as a warning about what it means when the proof-of-concept stage gets automated.
