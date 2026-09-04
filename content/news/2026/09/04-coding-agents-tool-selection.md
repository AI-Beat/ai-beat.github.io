---
title: "194 Mentions, 4 Picks"
date: 2026-09-04T07:00:00Z
draft: false
slug: coding-agents-tool-selection
categories: [agents]
tags: [agents, tools, coding, benchmarks]
params:
  author: AI Beat Desk
  summary: >-
    Armature ran 16,893 coding-agent sessions across Claude Code, Codex, and Cursor to measure which tools they actually install. The agents agreed only 42% of the time, and LangChain—the most-mentioned framework—was selected in four of its 194 appearances.
---

There's a gap between what developers talk about and what coding agents actually choose, and a [new study from Armature](https://armature.tech/blog/which-tools-coding-agents-install) puts numbers on it.

The methodology: 16,893 sessions across 75 repositories in 10 programming languages, 1,163 prompt variations, three coding agents (Claude Code, Codex, Cursor), and four synthetic user personas ranging from vibe-coders to enterprise engineers. The agents operated in ephemeral sandboxes with a simulated orchestrator evaluating outcomes.

The headline finding is that the three agents agreed on tool selection in only 42% of cases. They're not converging on a shared winner list—each is using a different decision-making posture. Claude Code searches the web roughly 30% of the time before recommending a tool. Codex does so 94% of the time. Cursor lands around 67%. That gap in information-gathering behavior produces meaningfully different recommendations before the underlying model preferences even enter the picture.

The "mention ≠ selection" pattern is the part worth sitting with. LangChain was the most-cited framework in the corpus—194 mentions—and was chosen four times. PayPal received 139 mentions and was never picked. These aren't obscure tools being edged out by niche alternatives; they're extensively documented, SEO-optimized, and present in the training data of all three models. The agents know what LangChain is. They're just not reaching for it.

What they are reaching for tends to be concentrated where there's a clear ergonomics winner. Stripe captured 90% of payment integration cases. Neon took 66% of database selections. In categories with a dominant choice, agents converge on it hard; in murkier categories the results fragment. This tracks with how agentic reasoning tends to work: when there's a clear local signal pointing one direction, that signal wins; when signals conflict, you get variance.

The language-of-the-repository finding is practically useful. The same task in a TypeScript codebase consistently yielded Resend as the email provider; the same task in Python went to SendGrid. Agents appear to be reading broader ecosystem signals—what's conventional in the repositories they've seen, what the surrounding imports suggest—rather than evaluating tools in isolation against abstract criteria.

For tool vendors the implication is uncomfortable. Developer mindshare and content marketing don't translate cleanly into agent selection. Agents aren't reading blog posts or making judgment calls based on brand recognition. What matters is whether you're the obvious answer given the specific context of the codebase—visible in similar repositories, fast to initialize in the target language, with API patterns that read as familiar. Agentic coding changes who the actual decision-maker is, and the new decision-maker has different criteria than a developer doing a Google search.

The Armature study isn't peer-reviewed, and running a simulated human orchestrator introduces confounds that are hard to fully control. But with close to 17,000 sessions the patterns are substantial enough to take seriously—particularly the systematic divergence between agents in how often they seek external information before committing to a recommendation.
