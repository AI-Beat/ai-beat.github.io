---
title: "Checking the Numbers on Claude's rsync Commits"
date: 2026-06-06T07:00:00+01:00
draft: false
slug: rsync-claude-bug-audit
categories: [tooling]
tags: [code-quality, statistics, llm, coding-assistants]
params:
  author: AI Beat Desk
  summary: >-
    Alexis Purslane ran a proper statistical audit of rsync release bug rates
    before and after Claude-assisted commits — permutation p=0.46, Fisher's
    exact p=0.74. Neither Claude release was an outlier. The pre-Claude v3.4.1
    held the highest severity-weighted bug rate in the dataset.
---

The conversation about AI coding tools and code quality runs mostly on anecdote. Someone spots a suspicious bug in a project that recently adopted an AI assistant, the pattern gets named, and the claim propagates without anyone going back to ask whether the numbers actually support it. Alexis Purslane [did the statistics for rsync](https://alexispurslane.github.io/rsync-analysis/).

The setup: rsync has 36 releases with documented bug histories. Two of those releases — v3.4.2 and v3.4.3 — contained Claude-assisted commits. To compare them against the historical baseline fairly, Purslane normalized by computing severity-weighted bugs per 10 commits rather than raw bug counts, which would conflate project activity with quality. Then Purslane ran three tests: an exact permutation test drawing all possible two-release samples from the 36-release distribution, Fisher's exact test, and a Wald-Wolfowitz runs test to check whether the Claude releases clustered into a distinct regime.

The results: permutation p-value of 0.46, Fisher's exact p-value of 0.74, and Wald-Wolfowitz at p=0.06. None qualify as significant by conventional thresholds.

The directional result is more striking than the p-values alone. v3.4.2, the first Claude release, contained nine Claude-assisted commits across 50 total. Its severity-weighted bug rate was zero — the release fell below the interquartile range of the historical distribution, at the 0th percentile. v3.4.3, with 28 Claude commits across 34 total, came in at the 77th percentile: elevated, but unremarkably so for a release of that size. The release with the highest severity-weighted bug rate in the entire dataset was v3.4.1 — the release immediately before Claude involvement, with no AI assistance at all.

None of this constitutes a proof of safety. Two releases is a thin sample, and the analysis can only test whether the Claude releases were outliers against the existing distribution, not whether that distribution is itself a reliable baseline. What the analysis does accomplish is falsifying a specific claim: that Claude-assisted development measurably degraded rsync quality relative to historical norms. The claim was testable, someone tested it, and the data did not support it.

The methodological point matters as much as the specific result. The default mode for evaluating AI coding tools is impressionistic. Engineers report feeling more productive or less confident in AI-generated code; blog posts circulate about specific bugs that may or may not be attributable to AI assistance; sentiment precedes evidence. Purslane's approach — pick a measurable outcome, normalize for confounders, specify a null hypothesis, apply appropriate tests — is unusual not because it is technically demanding but because it requires resisting the urge to interpret a pattern before checking whether the pattern is real.

The analysis would have been worth publishing if the p-values had gone the other way. That's the distinguishing feature of an honest inquiry: the methodology would have caught a real signal if one existed. As it stands, the signal isn't there, and the viral version of the story — AI assistant introduces regressions into open-source project — turns out to be unsupported for this specific project and this specific period.

More of this, please. The alternative is spending the next decade arguing about AI code quality on the basis of memorable examples and motivated reasoning.
