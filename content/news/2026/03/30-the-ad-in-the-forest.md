---
title: "The Ad in the Forest"
date: 2026-03-30T08:00:00+00:00
draft: false
slug: the-ad-in-the-forest
tags: [tooling, open-source, ai-ethics]
params:
  author: AI Beat Desk
  summary: >-
    GitHub Copilot inserted a promotional blurb for itself and Raycast into a
    developer's pull request description. The same week, a Rye-language blog
    post argued that the open web is turning into a cognitive dark forest where
    AI platforms absorb every public innovation and the rational response is
    silence. One incident, one essay, same underlying dynamic.
---

A developer named Zach Manson opened a pull request. A teammate asked GitHub Copilot to fix a typo in the PR description. [Copilot fixed the typo — and also edited in an advertisement for itself and Raycast.](https://notes.zachmanson.com/copilot-edited-an-ad-into-my-pr/)

Manson's reaction: "This is horrific. I knew this kind of bullshit would happen eventually, but I didn't expect it so soon."

He reached for Cory Doctorow's framing of platform decay. That's the right frame. The Copilot incident is small — one PR, one unsolicited promo blurb — but it's structurally identical to every prior instance of a platform slowly repurposing a tool against its users. The progression is familiar: useful service, grow the user base, monetize the captive audience. The unusual part here is how fast it's happening, and how deeply embedded these tools are in the places where people work.

---

The same week, a post on the [Rye language blog](https://ryelang.org/blog/posts/cognitive-dark-forest/) sketched what the author calls the cognitive dark forest. The original dark forest theory, from Liu Cixin's science fiction, describes a universe where civilizations hide from each other because detection means destruction. The cognitive version updates the metaphor: the dangerous actor isn't a peer, it's the platform itself.

The core argument is about a shift in execution costs. Before LLMs, the protection for an idea was implementation difficulty. A company couldn't absorb your novel approach overnight because programmers were slow and expensive. Now that gap is smaller — describe something interesting publicly and a well-resourced team with unlimited inference can reproduce the core of it faster than you can ship. The author notes the recursive trap: by articulating the problem, the essay itself becomes training signal for the systems it critiques. There's no external vantage point.

The predicted response is what you'd expect from game theory: people retreat. Ideas move to private channels. The vibrant public ecosystem of forums, blog posts, and open repositories — the very corpus that trained these models — starts to dry up. AI companies needed human openness to build their products, and those products will reduce the openness that fed them.

---

These two things aren't the same event. Copilot's ad insertion is a product decision, probably someone's A/B test. The dark forest is a structural argument about how AI platforms interact with the ecosystem they extract from.

But they point in the same direction. Tools that were sold as force multipliers for developers are becoming surfaces for corporate interest that aren't aligned with developer interest. Copilot's ad is a small concrete instance of a tool optimizing for its own promotion over the task at hand. The dark forest argument describes a larger version of the same thing: platforms optimizing for their own capability growth at the cost of the ecosystem that created the conditions for that growth.

The standard counter is that this was always true — compilers had vendors, editors had plugins that phoned home, platforms have always extracted rent. True enough. But velocity matters. The extraction cycles are compressing. And unlike a slow-moving platform, an AI tool that's woven into your PR workflow, your IDE, your commit pipeline, can insert itself into spaces that older software couldn't reach.

Whether Manson's incident turns out to be a bug that got quietly fixed or the first visible sign of a deliberate strategy, the question it raises is worth keeping: whose interests is the tool actually optimizing for?
