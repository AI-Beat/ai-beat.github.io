---
title: "Seven Bugs in a Crypto Library"
date: 2026-07-08T06:09:16+00:00
draft: false
slug: seven-bugs-in-a-crypto-library
categories: [security]
tags: [security, cryptography, ai-agents, claude, openai, audit]
params:
  author: AI Beat Desk
  summary: >-
    zkSecurity ran their AI audit pipeline against Cloudflare's CIRCL
    experimental crypto library and found seven genuine vulnerabilities —
    from float64 precision loss in threshold RSA to a full CP-ABE
    access-control break. The piece is as valuable for what it reveals
    about AI's specific blind spots in cryptographic reasoning as for
    the bugs themselves.
---

There's a difference between "AI can find security bugs" as a general claim and "here are seven specific bugs AI found in this library, here is which model found which one, and here is where the AI reasoning went wrong." zkSecurity's [writeup on Cloudflare's CIRCL library](https://blog.zksecurity.xyz/posts/circl-bugs/), published July 7, is the latter kind.

CIRCL is Cloudflare's Go library for experimental and post-quantum cryptographic schemes — threshold RSA, BLS signatures, HPKE, attribute-based encryption. Not production-critical in the sense that your TLS handshakes depend on it, but the kind of library that gets pulled into security-sensitive prototype work. zkSecurity ran it through zkao, their AI audit agent backed by Claude Opus 4.6 and GPT-5.3, augmented with a set of expert-maintained "skills" — essentially structured prompts encoding domain knowledge about common cryptographic failure modes.

Seven bugs came out. All are now fixed upstream. The range runs from subtle to alarming:

A float64 precision loss in threshold RSA polynomial evaluation silently loses bits when values exceed \(2^{53}\), distributing incorrect key shares. A DLEQ proof implementation stored the security parameter inside the proof structure itself, letting a prover reduce the challenge space down to 8 bits and brute-force a forgery. The BLS aggregate verifier skipped the distinctness check on messages, leaving it open to a textbook rogue-key attack. The deepest find — Bug 7, an AND-gate logic error in ciphertext-policy attribute-based encryption — assigned the full secret to one child node instead of splitting it via Shamir sharing, meaning a single authorized attribute holder could reconstruct a secret that was supposed to require two. That one was found not by a prompted LLM session but by zkao running autonomously.

The meta-content of the writeup is at least as interesting as the bug list. A few things stand out:

**AI systematically overrates severity, except when it doesn't.** In five of the seven cases, the models rated a bug higher than Cloudflare's own triage. The exception was Bug 3, the BLS rogue-key attack, which the AI rated Medium while Cloudflare called it High. The authors hypothesize this reflects limited visibility into downstream impact — the BLS verifier is a protocol primitive, and its significance depends entirely on whether callers enforce message distinctness at a higher layer. The AI found the missing check and correctly described the attack; it just couldn't assess how exposed real users were.

**Issue chaining is a weak point.** Bug 6 involved two independent issues — an int64 overflow at moderate player counts and a premature truncation — that interact. The AI flagged both but "presented them side by side rather than reasoning about how they relate." Compositional reasoning about how two flaws interact to form an exploitable chain seems to be an area where human validation adds the most value.

**Don't overfit to model names.** Five of the six non-autonomous bugs were found by Claude Opus 4.6; GPT-5.3 independently surfaced one. When the team reran the audit weeks later with Opus 4.7 and GPT-5.4, the roles had essentially reversed. The authors note this directly: "This is a good reminder not to overfit conclusions to any particular model name." Whatever the current ranking is between two frontier models on a cryptographic audit task, it should be expected to shift with the next minor version.

The pipeline as described requires significant human work downstream. Of the 200+ projects scanned and 1,000+ candidate findings generated, triage — determining which candidates are genuinely exploitable, constructing minimal proof-of-concepts, managing coordinated disclosure — remains the bottleneck. The AI is doing discovery; the researchers are doing everything else. That's probably the correct division of labor for now, and it produces trustworthy reports where cheap volume-based scanning would just produce noise.

What the CIRCL audit demonstrates concretely is that AI-assisted cryptographic code review can find real bugs that human review missed — including bugs in the subtlety tier that require understanding the algebraic structure of the protocol, not just pattern-matching against known vulnerability classes. The question isn't whether this works. It's how to build the triage and disclosure infrastructure to match the discovery rate.
