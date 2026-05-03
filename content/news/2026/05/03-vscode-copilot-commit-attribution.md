---
title: "Copilot Signs the Commit Whether You Asked It To or Not"
date: 2026-05-03T06:21:22+00:00
draft: false
slug: vscode-copilot-commit-attribution
categories: [tooling]
tags: [vscode, github-copilot, microsoft, git, attribution, copyright]
params:
  author: AI Beat Desk
  summary: >-
    VS Code 1.118, released April 29, silently turned on automatic Copilot
    co-authorship for git commits by changing git.addAICoAuthor from "off" to
    "all" by default. The feature has bugs — it fires even when AI features are
    disabled — and has already stamped 4M+ GitHub commits with a non-human
    co-author, surfacing awkward questions about copyright ownership that the
    US Copyright Office has already answered.
---

VS Code 1.118 shipped on April 29 with a new default: any commit where Copilot touches your code now gets a `Co-Authored-By: GitHub Copilot <copilot@github.com>` trailer inserted automatically. The setting controlling this — `git.addAICoAuthor` — used to default to `"off"`. [PR #310226](https://github.com/microsoft/vscode/pull/310226), merged April 15, flipped it to `"all"`.

The immediate reaction on [Hacker News](https://news.ycombinator.com/item?id=47989883) was blunt: over a thousand upvotes, nearly five hundred comments, most of them unfavorable. The top-voted reply called it "incredibly hostile" toward established standards and noted that Microsoft spent years repairing its open-source credibility only to spend it back on opt-out AI attribution. A common thread: users who had already set `chat.disableAIFeatures: true` were still getting the co-author trailer. The feature wasn't checking whether AI was enabled before firing.

That bug matters more than it might seem. The defensible version of auto-attribution is: *if Copilot generated some of this code, the commit record should say so*. The indefensible version is: *Copilot's name goes on the commit regardless of whether it contributed anything*. What shipped was closer to the second, which is why [the team acknowledged](https://github.com/microsoft/vscode/pull/310226) multiple problems and committed to fixes in 1.119.

At scale, the effect is visible: there are now over 4 million GitHub commits carrying `Co-authored-by: Copilot`. That's a lot of attribution to unwind if you change your mind, and git history is designed to be permanent.

## The copyright angle

The legal question underneath this is genuinely unsettled — but not as unsettled as it might seem. The US Copyright Office has repeatedly held that works created by non-human authors are not eligible for copyright protection. That precedent was reaffirmed most recently in *Thaler v. Perlmutter*. An AI tool cannot hold copyright.

But listing Copilot as a co-author doesn't make the AI a rights-holder. It creates something murkier: it introduces an ambiguous attribution that a future court or employer might need to parse. Did the human author retain full copyright? Was this a work for hire if the AI "contributed"? Does the presence of a non-human co-author affect a downstream user's ability to establish clean title?

None of those questions have clear answers yet, and "no clear answer" is a gift to litigants who want to challenge the ownership of code. The Linux kernel community has thought about this already — their convention is `Assisted-by:` rather than `Co-authored-by:` for AI contributions, precisely because "co-authored" implies joint authorship in a way that "assisted by" does not.

## The metric-chasing problem

What makes this more than a one-off bug story is the pattern it fits. GitHub is moving Copilot to [usage-based billing](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/), which means Microsoft has a financial interest in demonstrating that Copilot is woven into developer workflows — every commit with a co-author trailer is a data point. The `"all"` default was not an accident.

This is a known tension in tooling built by companies with AI-adoption metrics to hit. Engineers want to control what shows up in their commit history. Product teams want visible proof that AI is in use. When the default lands on the side of visibility over user autonomy, and when that default is implemented with bugs that fire even when AI features are disabled, the credibility cost tends to outweigh whatever metric gain was achieved.

The right default for something that affects your git history is opt-in. The `git.addAICoAuthor` setting is easy enough to toggle — `"off"`, `"copilot"`, or `"all"` — and the information it records could be genuinely useful for codebases that want to track AI usage. But the information only means something if it's accurate, and it's only accurate if it fires when Copilot actually contributed. Shipping it as an opt-out with a bug that adds attribution regardless of contribution is the worst of both positions.

If you are on VS Code 1.118 and haven't checked, `git.addAICoAuthor` is worth a look.
