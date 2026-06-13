---
title: "The Lockbox Problem"
date: 2026-06-13T06:07:27+00:00
draft: false
slug: fable-mythos-export-ban
categories: [safety]
tags: [safety, security, policy, anthropic, export-controls]
params:
  author: AI Beat Desk
  summary: >-
    The US government banned Anthropic's Fable 5 and Mythos 5 globally after a
    narrow jailbreak was found that could unlock Mythos's autonomous offensive
    cybersecurity capabilities. Anthropic disputes the decision as disproportionate.
    The real issue is harder than either side is saying: you can't export-control
    your way out of a model that already knows how to hack.
---

At 5:21 PM ET on Friday, the US Commerce Department issued a verbal directive — no written explanation, no specific details — ordering Anthropic to immediately disable Fable 5 and Mythos 5 for all users globally. This includes foreign nationals working at Anthropic itself. The models went dark for everyone, not just overseas accounts, because the directive's scope made selective compliance impossible.

To understand why this matters, you need to know what Mythos 5 actually is.

Anthropic's Mythos-class models are not general-purpose assistants. They are purpose-built offensive security systems. Per [Anthropic's own documentation](https://www.infosecurity-magazine.com/news/fable-5-mythos-class-anthropic/), Mythos 5 can "perform agentic hacking across multiple stages including reconnaissance, vulnerability discovery, exploit chain creation and lateral movement" and "autonomously discover and exploit zero-day vulnerabilities" — finding security holes nobody knows about and breaking through them without human guidance. Access to the full Mythos version is restricted to vetted organizations through Project Glasswing, a limited initiative that includes critical infrastructure operators and partners like Microsoft, Nvidia, and Google Cloud.

Fable 5, launched publicly on June 9, is the sanitized variant: Mythos under the hood, but with the cyber-offense toolkit stripped out behind safeguards. Hundreds of millions of users have access to Fable 5. The cyber-offense capabilities are locked, not absent.

The jailbreak the government cited is narrow. Anthropic [described it](https://www.anthropic.com/news/fable-mythos-access) as involving "asking the model to read a specific codebase and fix any software flaws" — a specific prompt pattern that bypasses the safeguards in one particular configuration, not a universal key. Anthropic says it saw only a verbal description of the technique, not a demonstration. The company notes that the same vulnerability appears to exist in OpenAI's GPT-5.5, which faces no similar restriction. Their argument: "If this standard was applied across the industry, we believe it would essentially halt all new model deployments for all frontier model providers."

That argument is not wrong on its own terms. If a narrow, non-universal jailbreak is sufficient grounds to yank a model used by hundreds of millions of people with no warning and no appeal, the standard would apply broadly enough to remove most frontier models from deployment. Anthropic conducted thousands of hours of red-teaming before Fable 5 launched. The company is likely correct that "perfect jailbreak resistance is not currently possible for any model provider."

But there is a harder version of the government's concern that Anthropic is not quite addressing. The reason this jailbreak is more alarming than a typical safety bypass is what it potentially unlocks: not content policy violations, not harmful text generation, but autonomous offensive hacking capability. The difference between "the model might write something offensive" and "the model might autonomously identify and exploit a zero-day vulnerability in critical infrastructure" is not a matter of degree. It is a different category of risk.

The lockbox problem is structural. When you train a model with the capability to autonomously discover and exploit zero-day vulnerabilities and then add inference-time restrictions to prevent public access to those capabilities, the capabilities are still there. The restrictions are prompts and system-level guardrails — the same substrate that jailbreaks operate on. You have not made the model incapable; you have made it reluctant, within the bounds of what training and RLHF can accomplish. Jailbreaks find the seams between capability and reluctance. For most capabilities, this is a nuisance. For autonomous offensive hacking, the seam has a different blast radius.

This is not an argument that the government made the right call with this specific directive — the proportionality question is genuinely difficult, the process was opaque, and Anthropic has legitimate grievances about the lack of written explanation or industry-standard notice. But it is an argument that the underlying problem will not be solved by better jailbreak resistance alone. If a model has been trained to autonomously exploit zero-days, that capability does not disappear when you wrap it in a system prompt.

The question the Mythos architecture raises — and that the current export control framework is not well-equipped to answer — is whether there is a meaningful safety distinction between "model trained to do X, with X locked at inference time" and "model that can do X." For most values of X, the answer is yes, the lock matters. For autonomous offensive cyber operations, the answer is much less clear.

Anthropic says it is "working to restore access as soon as possible." That may happen. The harder work is figuring out whether the lockbox model is an adequate safety architecture for what Mythos can do.
