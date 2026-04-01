---
title: "What the Source Maps Revealed"
date: 2026-04-01T06:15:00+00:00
draft: false
slug: claude-code-source-maps
tags: [tooling, claude-code, security]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic accidentally shipped source maps in their Claude Code npm package,
    exposing the full client-side source. The analysis that followed is worth
    reading not for the drama of a leak but for what the code reveals about
    the product's actual architecture: anti-distillation mechanisms, an "undercover
    mode" for employee contributions, and an unreleased background agent called KAIROS.
---

Source maps are a development tool that browsers use to map compiled JavaScript back to the original source, making debugging easier. They're supposed to be excluded from production builds. When Anthropic shipped their Claude Code npm package, they didn't exclude them — and the full client-side TypeScript source was available to anyone who looked.

[Alex Kim's analysis](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/) published March 31 walks through what the code actually contains. The piece is measured rather than sensationalist, which makes it more worth reading. The findings break into three categories.

## Anti-distillation machinery

The code includes mechanisms intended to prevent competitors from extracting training signal by recording API traffic. The implementation involves injecting fake tool definitions into the tool list that would "pollute" any training dataset constructed from captured conversations, along with server-side text summarization outputs that carry cryptographic signatures to detect tampering.

The author's assessment is that these protections are relatively easy to circumvent with basic filtering. That's probably right — if you're sophisticated enough to be building a competing model and recording API traffic, you can detect injected fake tools by cross-referencing against the actual tool schema. The countermeasures are more of a deterrent than a defense.

## Undercover mode

The more philosophically interesting finding is a feature called "undercover mode," which strips Anthropic-internal references from code written by Claude Code when operating outside Anthropic's internal repositories. The code comment notes explicitly that "There is NO force-OFF" for this feature, and that it's designed to prevent model codename leaks.

The practical consequence is that when an Anthropic employee uses Claude Code to generate or modify code in a public or external repository, the output is sanitized to remove internal identifiers. The feature exists for legitimate operational security reasons — you don't want internal codenames appearing in public commits — but the design choice of having no opt-out is worth noting. Code written by an AI system operating in undercover mode will appear in public repositories without disclosure of that fact.

This isn't an Anthropic-specific problem; it's a category of issue that will come up more broadly as AI coding tools are used by companies with internal terminology and proprietary context they don't want exposed. The solution space ranges from disclosure requirements to technical labeling, and nobody has settled on an answer yet.

## KAIROS

The most forward-looking piece of the analysis covers KAIROS, an unreleased autonomous agent mode visible in the codebase but not yet publicly available. The architecture is an always-on daemon that monitors repository activity and processes GitHub webhook events. The "autoDream" component runs memory consolidation when the user is idle — merging observations, resolving contradictions, and distilling vague notes into facts — so that when you return, the agent's context reflects everything that happened while it was running in the background. A separate feature flag, ULTRAPLAN, offloads complex planning to a remote container running Opus 4.6 with up to 30 minutes of thinking time.

This is a plausible direction for where coding agents are heading. The current model — you invoke the agent, it does something, you inspect the result — is interactive but synchronous. KAIROS as described is asynchronous: the agent runs continuously, receives events, and builds context over time rather than reconstructing it from scratch at each invocation. The nightly memory distillation component is particularly interesting because it addresses one of the real limitations of current coding agents, which is that their context is shallow relative to large codebases.

Whether KAIROS ships in anything like this form is unknowable from a source leak. Product roadmaps captured in source code are sketches, not commitments. But the direction is clear enough: Anthropic is building toward an agent that doesn't wait to be asked.

---

The accidental exposure itself is a routine operational security failure, the kind that happens when build tooling isn't configured correctly. The more durable consequence, as the analysis notes, is that the strategic intent behind features like KAIROS is now public in a way that can't be walked back. That's less about Anthropic specifically and more about what leaks always do: they collapse the timeline between "this is what we're building" and "this is what everyone knows we're building."
