---
title: "Knowing Isn't Doing"
date: 2026-09-08T06:11:59+00:00
draft: false
slug: agents-execution-gap
categories: [agents]
tags: [agents, benchmarks, robotics, evaluation]
params:
  author: AI Beat Desk
  summary: >-
    CivBench finds that agents told to check their progress every 20 turns do
    so every 30–75 turns, and follow through on only 48–66% of their near-term
    commitments. MIT's Phillip Isola published an essay the same week arguing
    that cloud LLMs are now ready to control physical robots. The juxtaposition
    is instructive.
---

Two things landed this week that sit in interesting tension with each other.

[CivBench](https://arxiv.org/abs/2609.02459), from a team at Oxford and Bristol, is a benchmark that runs language model agents through a full game of Civilization VI — 300+ turns, thousands of tool calls, incomplete information. The paper's key findings aren't about whether agents can play Civ; they can, roughly. The findings are about what happens between explicit instructions and actual behavior over a long horizon.

The authors measured two failure modes. The first they call under-monitoring: agents were told to check victory conditions every 20 turns. In practice they checked every 30 to 75 turns. In 7 of 20 observed defeats, the agent failed to query within the 20-turn warning window before the game ended. Not because it couldn't — the tool was available and the instruction was clear — but because it didn't. The second failure mode is what they measure with RAG@10: given a commitment the agent stated it intended to execute within the next 10 turns, what fraction of those commitments actually happened? Across models, this ranged from 48.2% to 65.8%. Roughly half to two-thirds follow-through on plans the agent itself articulated.

The paper's framing for this is careful: these are "deviations under instruction rather than absences of capability." The agents know the rules. They know the tools. They just don't apply them consistently over long stretches. This isn't a knowledge gap; it's a discipline gap.

---

Then, on September 7, MIT's Phillip Isola published ["Robot-Use Agents"](https://web.mit.edu/phillipi/www/writing/robot-use-agents.html), an essay arguing that frontier LLMs — he names Fable and Astra specifically — have crossed enough barriers that cloud-based LLM control of physical robots is now a credible path. The core argument: rather than developing device-specific AI for each robot form factor, you could have a cloud LLM act as a "puppeteer" for any device with a network connection. Existing robots could become AI-capable through software updates alone.

Isola is careful about the limitations. He notes the latency problem is real and the reliability is lower than traditional robotic systems. He frames these as surmountable rather than fundamental. But he also writes: "any device connected to the internet could soon become accessible to AI agents," and he seems to mean that as an observation about what's coming rather than a warning.

---

The thing the CivBench result puts a number on is exactly what you'd need to be confident about before deploying this kind of system. A 50% execution rate on stated commitments is fine for a game where losing a Civilization match has no consequences. It's not fine for a robot arm in a warehouse, a medical device, or any actuator in a context where the failure mode matters.

The CivBench authors are measuring in a relatively forgiving environment — the agent can miss a check and the game continues. The 7-in-20 defeat rate from missed monitoring windows is concerning precisely because the signal was explicit, the tool was available, and the failure was systematic across models, not an outlier.

Isola's essay and the CivBench paper are asking different questions, but they share a common object. He's asking whether LLMs can, in principle, operate robots. CivBench is asking how reliably agents actually follow through on their own plans under sustained pressure. The second question has to be answered satisfactorily before the first becomes safe.

The current numbers suggest there's real work still to be done between "capable enough to do the task" and "reliable enough to be trusted with the task." That gap isn't small, and it doesn't close on its own as models get better at reasoning — it requires a different kind of training signal, one that rewards consistent follow-through over time, not just correct action at each step.
