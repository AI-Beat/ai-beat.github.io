---
title: "The $100K Cryptanalyst"
date: 2026-07-29T06:11:36+00:00
draft: false
slug: claude-cryptanalysis-hawk-aes
categories: [research]
tags: [cryptography, security, research, agents, claude, post-quantum]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic published results showing Claude Mythos Preview semi-autonomously
    developed a novel attack on the HAWK post-quantum signature scheme — reducing
    its effective security by 26 bits — and a new fingerprinting technique on
    reduced-round AES that runs 200–800× faster than prior methods. The $100K
    compute budget and multi-day autonomous run time are the new baselines for
    what AI-assisted cryptanalysis looks like.
---

Anthropic [published results yesterday](https://www.anthropic.com/research/discovering-cryptographic-weaknesses) showing Claude Mythos Preview operating as a semi-autonomous research agent and producing novel cryptographic attacks on two targets. One of them — a key recovery attack on the HAWK post-quantum signature scheme — is significant enough that the HAWK developers received advance disclosure before the paper went public.

HAWK is a lattice-based digital signature scheme currently in the third evaluation round of NIST's Additional Digital Signatures process — the ongoing companion effort to the post-quantum standards that were finalized in 2024. Its main pitch is compactness: smaller signature and key sizes than the already-standardized ML-DSA (CRYSTALS-Dilithium), which makes it attractive for constrained environments where key overhead matters. The expected attack complexity for HAWK-256 was 2^64 operations. Claude found a key recovery attack that reduces this to 2^38 — cutting 26 bits of effective security. That's not a marginal rounding error; 2^38 is roughly a million times cheaper than 2^64, and while it's still not a laptop-scale attack, it meaningfully narrows the margin for long-lifetime deployments and challenges the original security claims. The finding doesn't kill HAWK outright (doubling key sizes would restore security), but it changes the calculus for anyone currently evaluating it against standardized alternatives.

The AES work is technically interesting but less immediately consequential. The target was 7-round reduced AES-128, a simplified variant that cryptographers study to understand how attacks might eventually scale toward the full 10-round cipher. Claude developed what Anthropic's paper calls a "Möbius Bridge" technique — a novel fingerprinting approach that improves a meet-in-the-middle attack by 200–800× depending on measurement method. Full AES-128 is unaffected; 7-round reduced AES has no production deployments. But the attack is new enough that it advances the known state of analysis on that construction.

The methodology is where things get interesting beyond the specific results. For the HAWK work, one Anthropic researcher spent roughly 60 hours guiding Claude through literature review and mathematical reasoning, with the model running most of the symbolic computation and verification work. For the AES attack, Claude ran nearly autonomously for about three days: it posed hypotheses, designed computational experiments to validate or refute them, and iterated toward the Möbius Bridge formulation. Total cost: approximately $100,000 in API calls.

That $100K figure invites two reactions. One is "expensive" — that's a meaningful compute budget, and it presumably reflects a lot of dead-end reasoning and wasted tokens before the productive paths emerged. The other is "cheap" — if the result genuinely advances cryptanalytic knowledge on a post-quantum candidate, it's probably cheaper than a PhD student stipend for the time horizon involved, and it doesn't require a human expert to hold the entire attack surface in working memory over months of part-time attention.

The useful comparison here is the [zkSecurity AI audit of Cloudflare's CIRCL library](https://www.ai-beat.net/news/2026/07/ai-crypto-audit-circl/), which we covered earlier this month. That was AI-assisted code review: the agent searched for known vulnerability classes in a real codebase, worked from a structured skill library of cryptographic failure modes, and found genuine bugs that human review had missed. Solid and useful. This Anthropic work is different in kind. "Möbius Bridge" isn't the name of a known attack template retrieved from a database — Claude appears to have named a technique it constructed. The target isn't implementation bugs but mathematical structure. That moves the capability profile from "AI as fast, thorough auditor of known patterns" to "AI as generator of new analytical methods," which is a different thing.

What this likely means for practice: standardization processes for post-quantum cryptography — NIST's, and whatever follows it — will need adversarial AI in the analysis pipeline the same way fuzzing became standard in software security testing. Lattice-based schemes have security assumptions that depend on subtle geometric properties, and the attack surface is mathematical rather than implementation-level. If a compute budget of $100K can shave 26 bits off a candidate scheme's security level, that's an analysis workflow worth running on every serious candidate, repeatedly, as AI capability improves.

What this doesn't mean: no deployed system is at risk from either finding, the full AES standard is fine, and the HAWK developers can respond with a key size adjustment. The threat model here is future deployments of HAWK in constrained devices where key size increases matter, and the multi-year planning horizon of post-quantum migrations. For those decisions, having better-than-expected attack complexity on a candidate scheme is useful to know now.
