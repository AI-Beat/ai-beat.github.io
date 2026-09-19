---
title: "The Decision Layer Eating the Agent Harness"
date: 2026-09-19T11:00:00+00:00
draft: false
slug: jev-agent-decision-layer
categories: [agents]
tags: [agents, inference, architecture, open-source, routing]
params:
  author: AI Beat Desk
  summary: >-
    Four days after TypeSafe AI launched Jev, 135+ public projects have
    appeared using it — and the pattern is clear: every routing and gating
    decision inside an agent harness is a perfect fit. With output tokens
    free and input at $42 per billion tokens, the economics of delegating
    those decisions are hard to argue with. Open-source clones are already
    circling, and the only thing they haven't reproduced yet is calibration.
---

The premise of Jev is almost offensively simple: strip a model of the ability to generate free-form text, train it specifically to return typed probabilities, and charge almost nothing for it. [TypeSafe AI launched it four days ago](https://typesafe.ai/blog/introducing-system-one-models-and-jev), and [we covered the architecture and framing then](/news/2026/09/typesafe-jev-system-one/). What's happened in the days since is worth its own piece.

Over 135 public projects have appeared in the [community-maintained awesome-jev list](https://github.com/yibie/awesome-jev) since launch. The breakdown is clarifying: 21 in the "agent decisions" category, 28 infrastructure projects (MCP servers, SDKs in Go, Rust, Swift, Elixir), 16 open replicas trying to reproduce the behavior locally. What keeps appearing is not the use case TypeSafe's marketing leads with — routing customer service emails, scoring business workflows — but something more specific: **every decision inside a coding agent harness**.

Consider what a coding agent actually does for most of its wall-clock time. It generates code, yes. But it also: decides which tool to call next, evaluates whether the previous tool output is sufficient, routes the request to a cheaper or more capable model tier, checks whether a proposed shell command is safe to execute, determines whether to invoke a full reasoning pass or proceed with what it has. None of these decisions are hard. They do not require paragraph-length reasoning. They are, in the language of the field, glorified function calls: structured state in, typed label out.

The [LangChain post on building a harness with Jev](https://www.langchain.com/blog/building-a-harness-with-jev) names this directly: "This was previously locked away in the closed-source parts of coding harnesses." Their AutoModeMiddleware pattern puts a Jev gate in front of every side-effecting tool call, blocking execution if confidence falls below a threshold and routing to a confirmation step otherwise. The call is under 200ms and costs fractions of a cent. The alternative — a full LLM evaluating the same decision — costs ten to a hundred times more and takes ten to twenty times longer.

[kevin9327's jev-harness](https://github.com/kevin9327/jev-harness) distills this to three outcomes: execute / confirm / reject, one Jev call per tool invocation. [JoacoMarc's jev-harness-router](https://github.com/JoacoMarc/jev-harness-router) goes further: a single batched Jev call at the start of each agent turn decides the model tier, which tools to expose, which skill to invoke, and the effort budget — four consequential routing decisions in one sub-500ms call, before the expensive model even sees the prompt. The economics of that trade are hard to argue with on [OpenRouter](https://openrouter.ai/typesafe/jev-1.13): $0.042 per million input tokens, output free. The reason output is "free" is that Jev's output isn't generated text — it's a probability distribution over a fixed set of options, and metering it apparently wasn't worth the billing infrastructure. That works out to $42 per billion input tokens, which is what the pricing looks like at production decision-making volumes.

The real-time applications are where the speed becomes concrete. The [typesafe-computer-use tool](https://github.com/awlevin/typesafe-computer-use), which surfaced on Hacker News this morning, automates macOS by OCR-ing the screen and calling Jev to classify the next action at $0.0002 per decision — the expensive model only engages for free-form text entry. At Vercel, replacing their LLM-based command safety classifier with Jev got them 5–18x faster results with better accuracy. Bryo AI's CTO tested Jev for email categorization against Gemini and found it 10–20x faster, with the key advantage being calibrated confidence scores: "the only one that hands back a real probability which makes it ideal for automating workflows."

That calibration point is the one thing the growing cluster of open-source clones can't reproduce. The approach itself is obvious enough that five independent implementations appeared within days of the launch: [OpenJev](https://github.com/yibie/awesome-jev) reads next-token logits directly from a frozen Qwen3.5-4B instead of running an autoregressive decode; [mini-jev](https://github.com/yibie/awesome-jev) takes a similar approach and achieves 0.909 accuracy on a standard multiple-choice benchmark against Jev's 0.907 across 6,750 observations — close enough to be useful. The MLX parallel constrained decoding project targets Apple Silicon with Qwen2.5-1.5B. None of them have published architecture specifics either, because TypeSafe hasn't published any — there is no paper, and RLCD (their training method) remains a name without a mechanism. What the clones miss is that logit rankings are not calibrated probabilities. A model that says "option B, score 0.73" where 0.73 is a ranking signal and a model that says "option B, confidence 0.73" where that means it's actually right 73% of the time behave identically in the output but completely differently when you use the score to gate autonomous execution.

Calibration is the hard part, and it's precisely what makes Jev's confidence score a useful routing signal rather than a rough preference. Without it, you can build a fast, cheap classifier; you can't build a reliable decision gate that hands off to a human at exactly the right threshold. TypeSafe's lead is real, but whether it's a durable moat depends on whether RLCD is the kind of method that can be reconstructed from first principles once people know it works, or whether it requires training data that only TypeSafe has.

Sixteen open replica projects in four days is a recognizable pattern: a new capability appears, its surface area is small enough that people can build toward it quickly, and the ecosystem fills in around the original within weeks. The coding harness integration is going to be table stakes for agent infrastructure before the end of the year. What remains genuinely unclear is whether the next Jev comes from TypeSafe, from a lab that publishes the training method, or from someone who figured out that calibration is achievable with a small labeled dataset and the right RL objective.
