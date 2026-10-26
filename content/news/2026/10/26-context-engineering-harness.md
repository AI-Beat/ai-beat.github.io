---
title: "The Context Stack Has Moved Downstairs"
date: 2026-10-26T06:13:46+00:00
draft: false
slug: context-engineering-state-of-field
categories: [agents]
tags: [context-engineering, agents, harness, prompting, engineering]
params:
  author: AI Beat Desk
  summary: >-
    Drew Breunig's essay on context engineering traces how the complexity that
    once lived in prompts has migrated into the infrastructure around the model.
    Long context windows didn't simplify things — they just moved the hard
    decisions into the harness, and trained agents introduce two new failure
    modes the harness can't fix.
---

Drew Breunig published [a clear-eyed essay on context engineering](https://www.dbreunig.com/2026/10/24/the-state-of-context-engineering.html) two days ago that's worth reading carefully if you're building anything with agents. The core argument is that what we've been calling "context engineering" has already become something else: harness engineering. The complexity hasn't decreased — it's relocated.

The framing is useful. In 2024, a certain view held that very long context windows would make retrieval-augmented generation and agentic scaffolding largely unnecessary. Give the model enough space and it could hold everything it needed. That view turned out to be wrong, and Breunig names four specific failure modes that explain why: *poisoning* (a document in context misleads the model), *distraction* (the model attends to irrelevant context when it should ignore it), *confusion* (conflicting information in context causes the model to hedge or err), and *clash* (two strong signals in context pull in opposite directions). None of these are solved by making the context window bigger. A larger window just means more of each.

The fixes developers reached for — augmentation, tool loadout, quarantine, pruning, offloading, summarization — were mostly known before the long-context era. What changed is where they live. They used to be managed in prompts, because the prompt was where you could intervene. Now they're managed in the harness — the scaffolding that decides what goes into the context window, in what order, with what preprocessing. The engineer who used to be a prompt writer is now writing a context assembly pipeline.

That's already happening and reasonably well understood. What Breunig adds is an analysis of two new failure modes that emerge when agents are trained to operate in harnesses rather than just operate with them.

The first is *fighting the weights*. Models trained extensively on agentic tasks develop strong priors about how to behave — they tend toward serial tool calls, for example, because that's what they've seen rewarded. If your harness wants them to parallelize or batch, you're working against the training distribution. The model has opinions. You have to argue with them via the prompt, and sometimes lose.

The second is *context inertia*. An agent pursuing a goal through a long sequence of steps will accumulate a working hypothesis about what's true and what to do next. Once that hypothesis is established, the model tends to continue refining it rather than revising it — even when new evidence in context should prompt a pivot. The agent keeps digging the same hole. The fix Breunig suggests is aggressive: clear the context, keeping only artifacts on disk, and let the agent restart with a clean slate. That's a surprisingly low-tech solution for a problem that sounds sophisticated, but it probably works.

The future section is speculative but pointed. Persistent agents — systems that accumulate memories and skills across sessions — will eventually carry so much context history that it starts to conflict. Different tools added at different times assume different things. Old memories contradict new ones. The management problem Breunig calls "strategic forgetting" is about deciding what to drop, and when, and by what criterion. It's a version of a problem knowledge management systems have always had, adapted for a context window.

The essay doesn't give you a recipe, but it gives you a vocabulary and a mental model, which is what you actually need when you're debugging an agent that behaves strangely at step 40 of a 50-step plan. Knowing that the problem might be context inertia rather than a bad prompt is half the diagnosis.
