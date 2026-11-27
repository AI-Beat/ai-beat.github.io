---
title: "The Ark Under the Router"
date: 2026-11-27T06:11:15+00:00
draft: false
slug: bytedance-ark-novita-streamlake
categories: [security]
tags: [security, privacy, inference, supply-chain, bytedance, china]
params:
  author: AI Beat Desk
  summary: >-
    An investigation published November 25 traced API requests from Novita and
    StreamLake — two inference providers presenting themselves as US-based with
    zero-data-retention guarantees — to ByteDance's Ark API running on Volcano
    Engine servers in Beijing and BytePlus servers in Singapore. Novita had already
    lost its ZDR label on OpenRouter five days before the post went live.
---

Five days after the [Dragos canary-token study](https://ai-beat.github.io/news/2026/11/inference-canary-tokens/) showed that inference provider ZDR claims were unreliable and that an unnamed Chinese lab was fetching URLs from more than half the prompts it received, an [independent investigation by kendell.dev](https://kendell.dev/blog/arkwrapper/) named some names.

The targets are Novita and StreamLake, two providers available through OpenRouter and other aggregators. The investigator traced actual API requests from both services to ByteDance's Ark platform — the company's commercial LLM inference API, sold as Volcano Engine inside China and BytePlus ModelArk outside it. ByteDance is the parent company of TikTok.

The evidence chain for Novita runs through several independent signals. Its DeepSeek V4.1 Flash requests were traced to Volcano Engine servers in Beijing; DeepSeek V4 Pro 0813 requests were traced to BytePlus in Singapore — both captured around November 21. Novita's support confirmed that "Partner"-labeled models on its platform may route to Qwen and inclusionAI. The returned V4 Pro fingerprint matched ByteDance's 0813 checkpoint, which is the generally available version sold on Ark. And Novita quietly removed its ZDR designation on OpenRouter on November 20, one day before the investigation captured those traces.

StreamLake's trail is different but arrives at the same destination. Binary assets for V4.1 Flash are fetched from a ByteDance CDN server in Beijing. Some error messages from StreamLake's gateway are identical word for word to Novita's gateway errors — suggesting the two share infrastructure, and that at least some StreamLake traffic passes through Novita on the way to ByteDance. Tool-call IDs from StreamLake's V4.1 Flash follow Ark's format rather than the standard OpenAI format. When asked about the Tiananmen Square tank photograph, V4 Pro via StreamLake gives the same canned refusal that ByteDance's models give — a consistent behavioral fingerprint.

None of this means every Novita or StreamLake request goes to Beijing. Routes change; not every model at either provider is a ByteDance resale. The investigation is a November 21 snapshot, and the researcher discloses that the work received a grant from OpenRouter, which is a competitor. Both caveats are worth noting.

What makes the investigation useful beyond the November 21 findings is the method. The Dragos canary study measured *outcomes* — whether seeded secrets were accessed. This one measures *routing* — where requests are physically sent before they return. The two approaches are complementary. You can have a provider that never leaks your canary tokens but still routes computation to infrastructure you weren't told about. You can also have a provider that routes cleanly but breaches stored data. The combination gives a more complete picture of inference supply-chain risk than either alone.

The practical question for anyone who chose Novita specifically for its ZDR label is straightforward: that label was removed five days before this investigation published, and the investigation found that requests were already going to Volcano Engine before the removal. The ZDR designation was not protecting what the user might have thought it was protecting.

The broader inference provider market has about fifty routing layers between user request and actual model compute, and the entities operating those layers are not uniformly disclosed. A developer building an application that handles regulated data, proprietary IP, or anything subject to cross-border transfer rules needs to treat "where does the compute actually happen" as a question that requires verification, not one that a terms-of-service page answers.
