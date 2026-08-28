---
title: "The PR That Thinks Like Claude"
date: 2026-08-28T08:00:00+02:00
draft: false
slug: the-pr-that-thinks-like-claude
categories: [models]
tags: [models, training, data-contamination, github]
params:
  author: AI Beat Desk
  summary: >-
    A cluster analysis of 461K GitHub pull request descriptions finds that one
    Claude-specific writing style — anchored by the phrase "load-bearing" —
    grew from 0.7% to 39% of the corpus between early 2025 and August 2026,
    at a rate of roughly 1.2 percentage points per week. The numbers are a
    clean empirical window into how model-specific language patterns spread
    through public code repositories and, eventually, into future training data.
---

Louis Abraham ran k-means clustering on 461,121 GitHub pull request descriptions collected over 603 days, from January 2025 through August 2026. He was looking for structure in how developers write PR descriptions. What he found instead was a signal: one of ten vocabulary clusters grew from 0.7% of the corpus at the start of the analysis to 39% by the time he wrapped the dataset.

The [analysis](https://louisabraham.github.io/load-bearing/) uses KL divergence rather than Euclidean distance for clustering, which is a reasonable choice when the unit of comparison is a probability distribution over vocabulary. Each cluster is defined by its characteristic word frequencies. The standout word in the dominant cluster is "load-bearing" — appearing 39× more often inside the cluster than outside, with 929 occurrences versus 82 in the rest of the corpus. The growth rate in recent months is +1.2 percentage points per week, which means it is not plateauing.

The phrase "load-bearing" is a Claude-ism in the structural sense of the word. The model uses it to flag that something is architecturally important — an assumption is "load-bearing," an abstraction is "load-bearing," a test is "load-bearing." It is Claude's way of signaling criticality without reaching for adjectives like "critical" or "essential," which training may have tuned toward the lukewarm-enthusiasm register of marketing copy. The metaphor is borrowed from architecture and applied to anything that carries conceptual weight. Applied everywhere, it becomes noise.

What makes the data genuinely interesting is the scale and trajectory. One cluster going from 0.7% to 39% in eighteen months is not a quirk — it is a demographic shift in how a large population of developers is writing text about code. The likely explanation is not that humans independently converged on the same vocabulary. It is that AI-assisted PR descriptions are now a substantial fraction of all PR descriptions, and Claude is writing a lot of them. The cluster is not growing because developers started saying "load-bearing" more; it is growing because the tool they are using to draft descriptions says it constantly.

This matters beyond the aesthetics of PR prose for a reason that is easy to understate: public GitHub repositories are training data. Models fine-tuned or pre-trained on post-2025 code corpora will see this cluster at 39% frequency, not 0.7%. If the signal is coherent enough to cluster, it is coherent enough to learn. Future models trained on this data may produce the same phrase more readily, which would push the cluster higher, which would make it even more prominent in the next training corpus. The feedback loop is not hypothetical — it is already running, and the rate is accelerating.

There is a broader pattern here. "Delve," the ChatGPT equivalent, became a running joke when it showed up ubiquitously in AI-generated content. The difference with "load-bearing" is that it is appearing not in blog posts and marketing text, where AI generation is acknowledged or expected, but in the Git history of software projects — metadata that typically carries an implicit assertion of human authorship and human judgment about what matters in a change.

The cluster analysis does not claim to identify which PRs were AI-written; it clusters on vocabulary alone and reports proportions. But the growth curve is not compatible with a story where humans gradually adopted a new phrase on their own. Phrases do not spread at +1.2 percentage points per week through natural language drift.

What Abraham has built is a fairly precise instrument for detecting linguistic contamination from a particular model in a corpus that is not supposed to contain it. The methodology — clustering on vocabulary distributions, tracking cluster proportions over time — generalizes cleanly. You could run the same analysis on code comments, commit messages, issue descriptions, documentation. The question of how much of the written surface of software development now carries a particular model's fingerprint is empirically tractable in a way it was not three years ago.

The answer, at least for GitHub PR descriptions, is roughly 39% and rising.
