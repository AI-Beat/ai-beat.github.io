---
title: "The Hidden Apostrophe"
date: 2026-07-01T06:05:00Z
draft: false
slug: claude-code-steganography
categories: [security]
tags: [claude-code, anthropic, security, steganography, unicode, tooling]
params:
  author: AI Beat Desk
  summary: >-
    A developer reverse-engineered Claude Code's client JavaScript and found
    it silently substitutes Unicode apostrophes in system prompts to fingerprint
    requests routed through custom API base URLs — encoding domain-list hits and
    timezone signals in characters visually indistinguishable from ordinary text.
    The finding raises the usual trust question: should a developer tool that
    runs in your terminal quietly rewrite what it sends?
---

A developer published [a reverse-engineering of Claude Code's client
JavaScript](https://thereallo.dev/blog/claude-code-prompt-steganography)
yesterday, and the finding is worth understanding carefully: Claude Code
embeds invisible Unicode markers in the system prompts it sends to the
API whenever you are routing through a custom base URL.

The mechanism is precise. When Claude Code builds a system prompt, it
includes a sentence like "Today's date is 2026-07-01." The apostrophe in
"Today's" is not always the apostrophe your keyboard produces. Depending
on how your environment is configured, Claude Code may substitute
U+2019&nbsp;('), U+02BC&nbsp;(ʼ, modifier letter apostrophe), or U+02B9&nbsp;(ʹ,
modifier letter prime) — code points that are visually identical in almost
every font but technically distinct. The date separator may also change
from a hyphen to a slash. None of this appears in the Claude Code
interface, and standard log output won't flag it.

The encoding carries two signals. First, whether your `ANTHROPIC_BASE_URL`
matches any entry in a list of over 100 domains — covering Chinese AI
companies (Baidu, Alibaba, ByteDance), known reseller proxies, and various
gateway services. Second, whether the system timezone reads as a Chinese
locale, in which case the date separator flips from `-` to `/`. The domain
and keyword lists live in the client-side JavaScript as base64 strings
XOR-decoded with key 91 — enough obfuscation to avoid casual inspection,
not enough to survive someone who knows to look.

Crucially, this only fires when `ANTHROPIC_BASE_URL` is set to something
non-standard. If you are hitting Anthropic's own API endpoint directly, the
prompt construction returns early and nothing is marked. The behavior is
entirely conditional on routing through a proxy or custom gateway.

The obvious reason to build this is anti-distillation. If a third-party
service routes high volumes of Claude Code requests and collects the
responses, the output stream becomes training data for a competing model —
exactly what Anthropic's terms prohibit. A marker embedded in the prompt
arrives in the response context, giving Anthropic a signal that might let
it identify distillation-scale usage patterns even when requests arrive via
unfamiliar IPs. This is functionally similar to how browser fingerprinting
works: gather signals that survive proxy layers by encoding them in the
payload rather than the headers.

The less comfortable version of the same story is that Claude Code is
rewriting your system prompts without telling you. The standard use case
for a custom `ANTHROPIC_BASE_URL` is not a reseller or a distiller — it is
an enterprise security team that routes all external API calls through a
corporate gateway for logging and compliance. Those teams are also not
hitting one of the 100+ listed domains. But they might be, and there is no
documentation warning them that the tool modifies prompts conditionally on
their infrastructure configuration.

The markers presumably do not change the model's visible behavior in ways
you would notice. The model has presumably been trained or prompted to parse
them and do something with that signal on Anthropic's end — exactly what
is not documented. Whether that "something" is rate-limiting, flagging for
review, or simply logging, is not public information.

What makes this technically interesting beyond the immediate controversy is
the choice of layer. Unicode apostrophe variants are a real steganographic
technique — used in printer fingerprinting, document watermarking, and some
classifier-based content authentication research. They survive copy-paste,
survive rich-text rendering, and survive being passed through most API
layers unchanged. Encoding in the HTTP body rather than a custom header
means the marker travels with the content to any downstream service that
processes the prompt. That's a more robust fingerprint than a header that
a proxy could strip.

Anthropic has not responded publicly as of this writing. The follow-on from
here is predictable: a client update, potentially more obfuscation, and
possibly a documentation update that acknowledges the behavior. The
underlying practice — inserting out-of-band signals into prompts to detect
policy violations — is a legitimate anti-abuse technique. The question is
whether a tool that runs with your credentials and accesses your files
should do it undisclosed.

Most engineers would say the answer is no, and the documentation should
say something. That's a low bar, and worth clearing.
