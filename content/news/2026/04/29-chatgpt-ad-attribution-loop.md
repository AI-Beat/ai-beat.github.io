---
title: "OpenAI's Ad Stack, From the Inside"
date: 2026-04-29T06:25:00Z
draft: false
slug: chatgpt-ad-attribution-loop
categories: [industry]
tags: [openai, chatgpt, ads, attribution, tracking]
params:
  author: AI Beat Desk
  summary: >-
    A technical reverse-engineering of ChatGPT's ad delivery system shows
    how OpenAI injects ads directly into the SSE conversation stream and
    closes attribution via four Fernet-encrypted tokens and a merchant-side
    JavaScript SDK — a fully first-party ad stack that bypasses any third-party
    intermediary.
---

OpenAI announced advertising in ChatGPT back in February. Since then, the company has been quiet about how it actually works. A [detailed technical writeup](https://www.buchodi.com/how-chatgpt-serves-ads-heres-the-full-attribution-loop/) published yesterday reverse-engineered the full system from the browser, and the architecture is more carefully designed than most people probably assumed.

The starting point is delivery. Ads don't arrive via a separate ad call or a page reload. They show up as structured `single_advertiser_ad_unit` objects injected directly into the SSE stream at `chatgpt.com/backend-api/f/conversation` — the same endpoint that delivers the model's text output, token by token. The ad objects are interspersed with model tokens as the response generates. Creative assets — images, favicons — are hosted on `bzrcdn.openai.com`, not merchant servers, which means OpenAI observes every impression without relying on merchant-side pixels.

Each ad carries four Fernet-encrypted tokens:

- `ads_spam_integrity_payload` — sent inside the SSE data, never on the click URL; used server-side to validate that clicks weren't forged
- `oppref` — the forward attribution token; persisted as the `__oppref` first-party cookie with a 720-hour TTL after a click
- `olref` — outbound link reference, paired with `oppref` but not retained by the tracking SDK
- `ad_data_token` — additional Fernet material for server-side reconciliation

Fernet is symmetric authenticated encryption (AES-128-CBC + HMAC-SHA256). OpenAI holds the decryption keys; advertisers and users don't. There's one observable exception: Fernet's first nine bytes are always public — a version byte plus an 8-byte timestamp — so you can read the creation time of any attribution token without any keys.

When a user clicks an ad, the landing URL carries `?oppref=` with the encrypted token. A JavaScript SDK, `oaiq.min.js`, running on the merchant's site reads it and writes it to `__oppref` cookie. Subsequent conversion events — add to cart, purchase, whatever the merchant configures — POST to `bzr.openai.com/v1/sdk/events` with that identifier. Attribution closes server-side at OpenAI.

The targeting observed in the writeup is contextual: the test account received travel ads during Beijing trip-planning conversations, sports betting ads during NBA discussion, and productivity tool ads in a project management context. No apparent cross-session profiling; the match is to the current conversation topic.

What makes this architecture worth studying is the structural choice to go fully first-party. There's no Google Ad Manager in the loop, no Meta Pixel, no third-party exchange. OpenAI controls the impression (they own ChatGPT), the creative hosting (their CDN), the click tracking (their SDK running on merchant sites), and the conversion measurement (their backend). The only party that can observe the full funnel is OpenAI.

This isn't just about privacy. It's about monetization leverage. Every major advertising platform — Google Search, Meta, TikTok — eventually ends up in a tug-of-war with advertisers over attribution accuracy. Advertisers want to verify numbers independently; platforms have an obvious interest in favorable measurement. OpenAI is building its stack in a way that maximizes its own observability, with merchants running OpenAI's code on their pages and POSTing their conversion data to OpenAI's servers.

From a user's perspective: clicking an ad writes a tracking cookie regardless of any consent flow, and the merchant-side SDK is on a page you've navigated to, so it runs at domain authority. Opting out means not clicking ads — there's no signal you can send to opt out of attribution if you engage.

None of this is surprising for an ad platform, exactly. But it's notably cleaner than most legacy ad infrastructure, which accreted over decades into a tangle of third parties, tag managers, and competing measurement claims. OpenAI built something tightly integrated from the start. The question worth watching is how much of that advantage holds as regulators and advertisers start asking harder questions about measurement independence.
