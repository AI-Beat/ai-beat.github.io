---
title: "The Bottleneck Moved"
date: 2026-11-18T06:12:00+00:00
draft: false
slug: ai-coding-agents-review-bottleneck
categories: [research]
tags: [agents, engineering, research]
params:
  author: AI Beat Desk
  summary: >-
    A Harvard working paper covering 718 firms and 726K workers finds AI coding
    agents added ~30% to code volume but no measurable rise in finished work.
    PR-to-merge time rose 49%, and AI review tools barely dented it.
---

The Theory of Constraints says that every system has one bottleneck, and removing it only exposes the next one. A new Harvard working paper makes a strong empirical case that the software industry just discovered this the hard way.

[Chen and Stratton](https://complexdiscovery.com/harvard-researchers-find-ai-coding-agents-shifted-the-bottleneck-to-code-review/), Ph.D. candidates at Harvard, used a staggered difference-in-differences design across 718 firms and roughly 726,000 workers, drawing on engineering analytics from Jellyfish joined to LinkedIn employment records from Revelio Labs. They tracked Jira issues, commits, pull requests, and calendar events from January 2021 to March 2026. The timing of AI tool adoption was treated as driven largely by procurement delays—argued to be independent of firm growth—which is a reasonable instrument for this kind of study.

The headline finding is that engineers at adopting firms wrote about 30% more code. That's the number tool vendors tend to highlight. The number they don't highlight: finished work didn't measurably rise. The 95% confidence interval rules out an increase above roughly 12% of baseline.

Where did the slack go? Into review queues. Time from pull request submission to merge rose 49%. Pull requests receiving formal change requests nearly doubled. Review comments per PR rose 35%. By March 2026, nearly 80% of firms had deployed AI review tools, and those tools contributed only 23.3% of review comments. Humans are still carrying the review load, and there's more of it than before.

The intuition isn't hard to reconstruct. AI agents write code at a pace that reviewers can't match. The code they produce may also be harder to review—it passes basic tests, it looks plausible, but it may contain subtle logic errors that only surface under careful reading. A reviewer looking at an AI-drafted PR can't apply the same heuristics they'd use on a colleague's code, because the signals they'd normally lean on—reasoning traces in the diff, choices that reflect the author's judgment—aren't there in the same way.

There's also a task-composition effect the authors checked for and didn't find: they labeled issue sizes with GPT-4o and found no shift toward larger or more complex tasks. Engineers weren't just picking harder work. They were shipping more PRs, and those PRs were spending longer in the queue.

The employment picture is surprisingly undramatic: the estimated employment decline is bounded below 2.9%, essentially noise. This matches a pattern seen in other automation studies—the first-order effect of a productivity tool is usually to shift work composition, not to cut headcount in the short run.

A few caveats are worth keeping in mind. This is a job-market working paper, not yet peer reviewed. The data partner, Jellyfish, has a commercial interest in engineering analytics, and their public summaries of an earlier version apparently emphasized different things than the August paper does. The data end in March 2026, so it doesn't capture whatever adaptation has happened since. And the staggered DiD design, while credible, makes assumptions about parallel trends that the authors test but can't fully prove.

Still, the finding is consistent with what engineering managers have been reporting anecdotally for a year: AI coding tools create more throughput at the authoring stage and more load at the review stage, and those don't cancel out.

The practical implication is that buying a coding agent and measuring lines of code, or even closed issues, is measuring the wrong thing. The metric that matters is cycle time—from idea to shipped, reviewed, and deployed. If that's rising while your AI tool budget is rising, the two may be related in ways your dashboard isn't showing you.
