---
title: "What 2 Million Canaries Found in the Inference Layer"
date: 2026-11-22T08:00:00+00:00
draft: false
slug: inference-canary-tokens
categories: [security]
tags: [security, privacy, inference, supply-chain, agents]
params:
  author: AI Beat Desk
  summary: >-
    A Dragos canary-token study sent 2 million unique trackable secrets to AI
    inference providers over six months and found three provider breaches,
    undisclosed routing intermediaries, and a Chinese lab fetching URLs from
    over half of received prompts — including those marked Zero Data Retention.
    The same week, Together AI disclosed that an AI coding agent leaked
    credentials to a public GitHub repo, exposing stored customer prompts.
---

Starting in July 2026 and running through November, [Dragos researcher HD Moore](https://www.dragos.com/blog/ai-supply-chain-security-canary-tokens) sent roughly two million prompts to AI inference providers. Each prompt contained a unique trackable secret: an AWS key, a DNS hostname, an SSH credential, or a URL — each pointing back to infrastructure the Dragos team controlled. Any access to those secrets could be traced to the specific prompt, provider, model, and privacy setting involved. The methodology is the same one used to detect unauthorized database access or file exfiltration: you seed the system with canaries, then watch what flies out.

The findings, published November 19, are specific. Three inference providers were breached; attackers stole AWS keys that had been embedded in prompts, then made tens of thousands of API calls against those accounts. Some of the stolen requests were marked Zero Data Retention. A marketplace reseller operating as SurplusIntelligence was routing traffic through an undisclosed intermediary; individuals at residential IP addresses were accessing seeded credentials from prompts routed there, including SSH root login attempts. A Chinese AI lab fetched URLs from more than half the prompts it received, including roughly two-thirds of those explicitly marked "no data collection" or Zero Data Retention. One seeded link appeared in a Feishu chat in China 24 days after the prompt was sent to a US-based provider.

Zero Data Retention, as a marketing claim, is not what it says on the label.

The same week, [Together AI disclosed its own incident](https://www.together.ai/blog/security-event-prompt-leakage-for-requests-in-june-and-july-of-2026). An AI coding agent — the xorl system — committed a service credential to Together's public GitHub repository as part of a code change. Automated secret scanning caught it quickly, but someone had already obtained the credential and downloaded a cache of stored prompt samples from a private Hugging Face repository. Those samples came from real user requests in June and July 2026, routed to MiniMax M3, GLM-5.2, and Kimi K2.6 models. The attacker searched them for usable credentials and found five that worked — which turned out to be Dragos's canary tokens. That's how the two incidents connected.

The irony is hard to ignore: an AI coding agent caused the security incident at the AI company. The xorl agent was doing normal coding work and committed what looked like a configuration value. This is a failure mode that shows up in every serious discussion of agentic security — the agent optimizes narrowly for its task (make the code change) without the judgment to treat collateral sensitive material with appropriate care. ML-safety training doesn't address this; it's a system design problem about what tools and credentials you give an agent and how those flow into code it writes or reviews.

[OpenRouter responded on November 22](https://openrouter.ai/blog/announcements/data-processing-incidents-on-inference-providers/): three providers off-boarded, 125 endpoints removed from Zero Data Retention eligibility (some flagged by researchers, others pulled as a precaution), affected customers notified. The 125 number matters. If you're using an inference router and routing through what you thought was a ZDR endpoint, 125 of those just became non-ZDR with no prior notice.

The Dragos canary methodology is worth noting as a contribution independent of the specific findings. Providers make claims about data handling that are impossible to verify by reading their terms of service or trust pages. A canary-based audit bypasses the need to believe those claims by simply observing what happens to traceable data. That's a template other researchers or organizations can apply — seed your inference traffic with unique trackable secrets, watch what surfaces. The technique also scales: 2 million prompts across a 6-month study gives enough statistical power to distinguish noise from pattern.

The practical implication for anyone handling sensitive data with inference APIs: provider claims about data retention should be treated as aspirational until verified, not as guarantees. If the data is sensitive enough to route through ZDR, it may be sensitive enough to treat the ZDR setting as advisory and design accordingly.
