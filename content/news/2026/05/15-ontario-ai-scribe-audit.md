---
title: "Ontario's AI Scribe Problem Is a Procurement Problem"
date: 2026-05-15T06:30:00+00:00
draft: false
slug: ontario-ai-scribe-audit
categories: [safety]
tags: [healthcare, hallucination, evaluation, medical-ai, deployment, procurement]
params:
  author: AI Beat Desk
  summary: >-
    Ontario's auditor general tested 20 government-approved AI medical scribes
    and found that 60% recorded the wrong drug, 9 of 20 fabricated treatment
    plans, and 17 of 20 missed mental health details. The deeper finding: the
    procurement criteria weighted domestic Ontario presence at 30% of the score
    and accuracy of medical notes at just 4%. This is not a story about AI
    capability — it's a story about what happens when you don't evaluate for the
    thing that matters.
---

Ontario's auditor general released findings this week on the AI medical scribes currently in use by approximately 5,000 physicians across the province. The results are specific and striking: of 20 approved AI scribes tested, 60% recorded a different drug than what was actually prescribed. Seventeen of the 20 missed key mental health details in at least one of two test scenarios. Nine of the 20 fabricated information — generating suggestions for therapy referrals or blood tests that were never mentioned in the simulated appointment recording.

That last category is worth dwelling on. These weren't transcription errors, where a spoken word gets misheard. The systems were inventing clinical actions — interventions that a physician reviewing an AI-generated note might plausibly act on — that had no basis in the actual conversation. In a patient encounter, a fabricated blood test order is not a neutral error.

The health ministry's response is that there have been no known reports of patient harm associated with the tools. That's probably true, and also probably not the right frame. Physicians are supposed to review the notes before acting on them. The question is whether the review burden being placed on 5,000 physicians — effectively, "treat the AI output as a draft that requires full verification" — is being communicated clearly, or whether the existence of an approved-vendor list creates an implicit warranty of accuracy that doesn't reflect the underlying performance.

The more damning part of the [auditor general's findings](https://www.sootoday.com/ontario-news/ontario-auditor-general-finds-problems-with-medical-ai-transcription-tool-12271379) isn't the accuracy numbers. It's the procurement criteria that got these tools approved in the first place.

Thirty percent of a vendor's evaluation score depended on whether they had a domestic presence in Ontario. The accuracy of medical notes — the core function of a medical note-taking tool — contributed just 4% to the total score. Eleven vendors submitted no third-party audit reports. Five submitted no threat risk assessments or privacy impact assessments.

A tool that records the wrong drug 60% of the time but maintains a Toronto office would outscore a tool that transcribes perfectly but operates out of another province. That's not a technology failure. It's a specification failure. Someone decided that regional economic policy was six times more important than whether the notes were correct.

This is a pattern that recurs whenever AI tools get deployed through institutional procurement processes rather than technical evaluation. The buyers don't always have the expertise to evaluate what they're buying, so they fall back on proxy criteria they understand: certifications, domestic presence, vendor size, insurance coverage. Accuracy benchmarks require designing test scenarios, hiring evaluators, and knowing what "good" looks like. That's harder than checking a box.

The auditor's recommendation is relatively modest: implement IT controls requiring physicians to attest that they reviewed AI-generated notes before they're finalized in patient records. That's a reasonable procedural fix. But it doesn't address the upstream problem, which is that the approval criteria didn't require the tools to work before being approved.

Healthcare AI deployment has a specific challenge that distinguishes it from most software procurement: errors can be directly harmful, the end users (physicians) are trained to trust records, and the feedback loop for catching errors is noisy. A billing error shows up in audits. A transcription error shows up in a patient chart reviewed by someone who may assume the record is correct because it was produced by a certified system.

The Ontario audit is a useful specific data point in what's becoming a broader pattern of institutional AI deployment done backwards — capability deployed, then evaluated. The tools went to 5,000 physicians before anyone checked whether they got the drugs right.

The fix isn't to stop using AI scribes. Accurate transcription assistance is genuinely useful in clinical settings, and the staffing pressure on physicians is real. The fix is to require that accuracy evaluation happen before approval rather than after deployment. That's obvious in principle. It apparently wasn't obvious enough in practice.
