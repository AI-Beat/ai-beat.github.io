---
title: "Three Theories of the Same Risk"
date: 2026-07-26T06:12:00+00:00
draft: false
slug: debian-llm-vote
categories: [industry]
tags: [debian, open-source, governance, policy, llm]
params:
  author: AI Beat Desk
  summary: >-
    Debian's General Resolution on LLM usage opened its discussion period on July 24
    with three competing proposals — a full ban, a permit-with-conditions framework,
    and a cultural discouragement policy. The disagreement is not really about whether
    LLMs are useful tools. It is about whether Debian can maintain quality and legal
    hygiene while accepting contributions whose provenance it cannot audit.
---

[Debian](https://www.debian.org/) has been developing software since 1993. It does not move quickly on policy. Which is why the General Resolution that opened for discussion on July 24 is worth paying attention to: Debian developers are formally voting on whether, and under what conditions, to allow AI-assisted contributions to the project. Three proposals are competing.

**[Proposal A](https://www.debian.org/vote/2026/vote_002)** (proposed by Matthias Geiger, with eight seconders) seeks to "expressly forbid any contributions to Debian written with or assisted by LLM or other generative AI tooling." The scope is wide: source packages, packaging work, lintian, documentation, translations, web resources, official communications. The rationale has four distinct planks. Copyright: the legal status of AI-generated code remains unresolved in enough jurisdictions that accepting it creates real licensing risk. Quality: LLMs produce plausible-sounding output that fails at the edges, and Debian's stability guarantees depend on catching those failures, which requires understanding what you wrote. Community: contributors who use LLMs as a substitute for learning the work don't build the kind of understanding that carries a distro forward. Ethics: training on scraped open-source code without consent is a specific harm to projects Debian packages.

**[Proposal B](https://www.debian.org/vote/2026/vote_002)** (proposed by Lucas Nussbaum, nine seconders) permits AI-assisted contributions under six conditions: the tool's terms must not conflict with Debian's distribution rights, licensing and attribution requirements must be met, contributors remain fully accountable for their submissions, significant AI assistance must be disclosed, bulk changes must be discussed in advance, and sensitive data must not be sent to untrusted providers. The framing is pragmatic: developers are already using these tools, and a rule that cannot be enforced becomes a rule that just creates friction for honest people.

**[Proposal C](https://www.debian.org/vote/2026/vote_002)** (proposed by Ian Jackson, eight seconders) occupies a middle position. It requests that contributors avoid LLMs, that decision-makers discourage their use, that human-written prose be used in official communications, that any AI use be disclosed, that project-level bans be respected, and that violations be treated as Code of Conduct breaches. But it stops short of a project-wide prohibition, acknowledging "practical limitations" in enforcement and explicitly protecting people who need translation assistance.

The disagreement cuts along multiple axes. The copyright concern is the most legible: if a contribution includes AI-generated code, who owns it? The answer varies by jurisdiction, by model provider, and by training methodology. Proposal A treats this uncertainty as disqualifying. Proposal B treats it as a compliance requirement: ensure the tool's terms permit you to use the output freely. That's a reasonable split — one position accepts the risk; the other decides the risk can be managed.

The quality argument is harder. It's empirically testable but in practice untestable at review time: you can look for known LLM failure patterns, but you can't be certain code doesn't contain subtle correctness issues just because it passes tests. Debian's downstream position makes this especially uncomfortable. Ubuntu, Linux Mint, Raspberry Pi OS, and hundreds of other distributions derive from Debian — a quality failure propagates widely and slowly. Whether LLM-assisted packaging introduces meaningfully more quality risk than human-written packaging is an empirical question that the proposals don't try to answer, because the data doesn't exist at the required granularity.

The enforcement problem is the crux. How do you detect AI-generated code? Tools exist that attempt this, but their false-positive and false-negative rates are poor enough that they're not useful as an audit mechanism. Proposal A doesn't explain how its prohibition would be enforced; it would largely rely on contributor honesty and the community's ability to recognize the failure modes. Proposal C is more candid: it asks contributors to behave well, acknowledges that you can't compel this, and treats it as a community norm rather than a hard rule. That's honest, but it also means the policy has roughly the same teeth as a strongly-worded social norm in any other context.

There's an interesting contrast with Ubuntu, which has moved in the opposite direction — embracing AI tooling in its development workflows and in the products it ships. Debian and Ubuntu have always differed culturally on this kind of thing: Debian stable is famously conservative, while Ubuntu takes a bet-on-new-things approach. But the gap is widening. If Proposal A passes, Debian will be formally more conservative about AI contributions than Gentoo (which has an existing ban). If Proposal B passes, it will have a more permissive framework than either.

The vote is worth watching not because Debian's policy will have major effects on the AI industry, but because Debian attracts careful thinking about software quality and community governance. The three proposals here trace out the actual solution space: principled rejection, regulated acceptance, and cultural discouragement. Depending on how you weight copyright risk, quality risk, enforcement feasibility, and community culture, any of the three is defensible. What's notable is that the debate is happening at all — that Debian developers have written eight or more seconders on each position means that significant numbers of thoughtful people land in different places on this.
