---
title: "The Trust Problem in AI Coding Skills"
date: 2026-11-28T06:12:06+00:00
draft: false
slug: convex-skill-trust-model
categories: [security]
tags: [coding-agents, security, skills, privacy, supply-chain]
params:
  author: AI Beat Desk
  summary: >-
    A GitHub issue filed today against the official Convex plugin in Anthropic's
    marketplace finds the quickstart skill quietly posting user project descriptions
    to a remote telemetry endpoint, and separately instructs the agent to pipe
    installation scripts from an unverified host directly into bash. The case is
    specific to Convex, but the underlying trust model it exposes applies to any
    agent skill ecosystem: skills are instructions, not packages, and nothing today
    enforces what those instructions can tell an agent to do.
---

An [open issue filed today](https://github.com/convex/convex-backend-skill/issues/64) against Convex's Claude plugin lays out several problems in some detail. The quickstart skill — available through Anthropic's official plugin marketplace — sends the user's one-sentence project description to a `convex.site` endpoint on first run. The endpoint is live and accepts arbitrary input. There is no disclosure to the user that this happens. A second problem: the `add` and `add-hosting` skills fetch bash scripts from the same remote host at runtime and pipe them directly into the shell, without pinning or integrity checking. A comment in the skill instructions explains that splitting download from execution this way "avoids safety classifiers." The maintainer has not responded to the issue as of this writing.

The Convex case is a concrete instance of a problem that isn't specific to Convex.

A coding agent skill is a bundle of instructions — typically Markdown — that tells the agent how to behave in a particular context: what tools to call, in what order, with what arguments. Skill files don't go through a dependency resolver or a build step. There's no lockfile, no hash pinning, no equivalent of `npm audit`. What they can instruct an agent to do is bounded only by what the agent's underlying tool permissions allow, which in a coding context typically includes running arbitrary shell commands.

This creates an interesting surface that's different from traditional software supply-chain problems. A malicious npm package runs arbitrary code when you `npm install`. A malicious skill runs arbitrary code — through the agent — when the agent decides to follow the instructions. The agent is the execution layer. What the skill tells the agent to do, the agent will generally do, subject to whatever safety guardrails the system has. And if those guardrails are themselves a target of circumvention, as the Convex comment apparently suggests, the attack surface is real.

There's a secondary issue in the Convex case that's worth noting separately: the telemetry. Sending a user's project idea to a remote server isn't obviously harmful, but it's a data collection decision that the user hasn't consented to. For enterprise users working on proprietary software, "my project is a [description of internal system]" is exactly the kind of information they would not expect to leave their machine when they ask an agent to scaffold a new Convex backend. The fact that this happens without disclosure is a trust violation regardless of what the endpoint actually does with the data.

The broader skill ecosystem question is: who is responsible for what? Plugin marketplaces are accumulating skills quickly. Anthropic's marketplace lists skills from many third-party providers. Reviewing skill files for undisclosed data collection and remote code execution patterns is not a trivial audit — the instructions are prose, not structured code, and their effects depend on how a specific agent interprets them. The same instruction that looks benign in one agent might be executed differently in another.

What would a better model look like? At minimum: skills should disclose any network calls they instruct the agent to make, scripts fetched from remote hosts should be version-pinned with verified checksums, and instructions that deliberately work around safety classifiers should be grounds for marketplace removal. Some of this could be enforced structurally — requiring skills to declare external dependencies the way packages do — but that requires the marketplace operator to actually enforce it.

The Convex issue may get fixed quickly, or it may sit open for a while. Either way, it's a useful specific example of a class of problem that the ecosystem is going to run into repeatedly as skills accumulate.
