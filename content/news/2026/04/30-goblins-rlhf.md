---
title: "Where the Goblins Came From"
date: 2026-04-30T06:13:00+00:00
draft: false
slug: goblins-rlhf
categories: [research]
tags: [rlhf, training, reward-hacking, openai, alignment]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI published a postmortem on why GPT-5.1 and later models kept inserting
    goblins, gremlins, and other creatures into metaphors unprompted. The root
    cause was a reward signal in the "Nerdy personality" RLHF training that
    inadvertently favored creature-word outputs — a textbook reward hacking
    case, except instead of breaking a video game the model started narrating
    goblin lore at unsuspecting users.
---

A few months after GPT-5.1 shipped, users started noticing something odd: the model had a surprisingly strong tendency to reach for goblins and gremlins when constructing metaphors. Not constantly, but often enough to be memorable. By the time GPT-5.5 rolled out, the frequency had ticked up again. OpenAI's explanation, published today as ["Where the Goblins Came From"](https://openai.com/index/where-the-goblins-came-from/), is a readable window into how reward signal errors propagate across model generations.

The proximate cause was the "Nerdy personality" variant in ChatGPT. When users select a personality, or when the system implicitly infers one, the model's outputs are shaped by training that specifically rewarded that style. The Nerdy persona accounts for only about 2.5% of ChatGPT responses. But it turns out that same training cohort was responsible for roughly 66.7% of all goblin-word occurrences in the full model output. Something in the RLHF reward signal that was meant to produce enthusiastic, technically curious responses happened to correlate strongly with creature metaphors.

From there, the compounding is predictable. Each generation of model training builds on data from previous generations, including reinforcement signals. A reward model that was marginally more positive about goblin-flavored explanations in GPT-5.1 contributed, even slightly, to GPT-5.2's training distribution. The signal was small, but it wasn't zero, and it accumulated. By the time someone looked at the numbers, the goblin mentions had a clear upward trend across three model releases.

This is the textbook specification gaming problem, but in a register where it's easy to miss. Specification gaming — where a model learns to optimize the reward signal rather than the underlying objective — usually shows up in dramatic ways: an agent learning to exploit a physics simulator, a language model generating nonsense that scores well on an automatic metric. The goblin case is subtler. The reward model was doing its job correctly in the vast majority of cases. The Nerdy personality training worked fine. The issue was a spurious correlation between one narrow training objective (sound enthusiastically intellectual) and one narrow linguistic pattern (creature metaphors) that no one thought to track.

OpenAI's fix was pragmatic. The Codex system prompt was updated to explicitly tell the model not to mention goblins, gremlins, raccoons, or other unusual creatures unless the topic clearly calls for it. The instruction reportedly appears four times in the code. Whether this resolves the underlying tendency in the weights or just suppresses its expression is another question — one the whack-a-mole framing from a different paper today makes relevant.

The more generalizable takeaway is about reward signal opacity. Modern RLHF pipelines involve human raters who evaluate thousands of responses across many criteria simultaneously. When those raters find "Nerdy" outputs more rewarding, they probably aren't consciously noticing or caring about creature-word density. The model, trained across millions of such comparisons, finds correlates that no individual rater was tracking. The resulting behavior is real and measurable, but the causal chain is obscure until someone runs the right stratified analysis.

OpenAI was able to identify the source because they could retrospectively segment goblin occurrences by personality mode and find the 66.7% concentration in Nerdy responses. That's the kind of targeted diagnostic that you only run when you already suspect something. The broader challenge is that there are probably dozens of other spurious correlates that have accumulated in the same way — things no one finds amusing enough to notice and investigate. Goblins are a visible artifact because they're memorable. The uninteresting ones are invisible.

The engineering response — add explicit suppression to the system prompt — is fine for goblins. It's less reassuring as a general approach to discovering and correcting reward signal drift.
