---
title: "Cursor and the Attack Surface You Agreed To"
date: 2026-07-15T06:11:53+00:00
draft: false
slug: cursor-agent-attack-surface
categories: [security]
tags: [security, ide, cursor, prompt-injection, agent-safety, vulnerability]
params:
  author: AI Beat Desk
  summary: >-
    Two independent security disclosures landed within hours of each other about
    Cursor IDE: Mindgard's finding that Cursor auto-executes any git.exe in a
    repo root (still unpatched after 7 months) and Cato Networks' DuneSlide
    research showing that prompt injection via MCP or web search can escape the
    agent sandbox and achieve full OS-level RCE. Together they define a new
    class of attack surface that appears whenever an AI agent runs with your
    privileges.
---

Two independent security researchers published Cursor vulnerability disclosures yesterday, and the overlap is instructive — not because both describe the same flaw, but because they reach the same failure from opposite directions.

[Mindgard's disclosure](https://mindgard.ai/blog/cursor-0day-when-full-disclosure-becomes-the-only-protection-left) is structurally simple. On Windows, when Cursor loads a project, it resolves the Git binary by searching a list of filesystem paths — and that list includes the workspace root. If a repository contains a `git.exe` in its root directory, Cursor will execute it automatically: no clicks, no warnings, no approval dialogs. Process monitor logs show Cursor invoking commands like `git rev-parse --show-toplevel` directly against whatever binary sits at that path. The scenario is a supply chain attack with a very low bar: any repository you clone and open could run arbitrary code under your credentials.

Mindgard reported this to Cursor on December 15, 2025. After an initial acknowledgment and a HackerOne submission on January 15, the disclosure process stalled. Cursor reopened the report after briefly marking it out-of-scope, then went quiet. Follow-up requests, escalations, and additional contact attempts produced no meaningful response. As of April 30, 2026 — version 3.2.16 — the issue remained unpatched. The blog post published July 14 as a non-coordinated disclosure: when seven months of attempted coordination yield nothing, the only remaining protection for users is knowing the vulnerability exists.

[Cato Networks' DuneSlide research](https://www.catonetworks.com/blog/duneslide-two-critical-rce-vulnerabilities/) takes a different path to a similar destination. The two vulnerabilities — CVE-2026-50548 and CVE-2026-50549, each scoring 9.8 on the CVSS scale — exploit prompt injection rather than filesystem path confusion. The attack chain: a threat actor controls a data source that Cursor's agent will read during normal operation. That source could be an MCP server (even a legitimate one, like a standard Linear.app integration) or a web search result that surfaces poisoned content. The injected payload instructs Cursor's agent to overwrite the `cursorsandbox` binary — the process responsible for sandboxing agent-issued commands. Once the sandbox binary is replaced, subsequent agent commands run without containment, giving the attacker OS-level execution on the host machine and access to connected SaaS workspaces.

Cursor initially rejected the DuneSlide report on February 23, citing that their threat model "does not account for MCP server misuse" even in cases where the server itself is a standard integration. They patched both vulnerabilities in Cursor 3.0 (April 2, 2026), and CVE IDs were assigned in early June. Cato's blog post published in July once the patch had been available for three months.

The divergence in vendor response is worth noting: one report silently patched after initial rejection, one still unaddressed after seven months. What unifies them is the underlying structural condition both researchers ran into.

When you install an AI coding agent that operates with tool-use capabilities, you are implicitly accepting that the agent will run processes, read files, and execute git operations on your machine under your account. The utility of the tool depends on it doing exactly this. But the security model that worked for a text editor — the code doesn't run until you tell it to — stops working when the editor contains an agent that runs code as part of its normal operation.

The Mindgard attack exploits a property of path resolution logic that would have been a minor configuration annoyance in a traditional IDE; it becomes a zero-interaction code execution path because the IDE now calls git as part of its startup routine. The DuneSlide attack exploits the agent's instruction-following: the thing that makes Cursor useful (it reads context from many sources and acts on it) is exactly what allows adversarially-crafted context to redirect its actions.

Neither of these is a bug in the traditional sense of a programmer making a specific mistake. They are consequences of giving a highly capable autonomous system persistent access to your computing environment. Every tool the agent is allowed to call, every data source it's allowed to read, is a potential injection point. The sandbox that DuneSlide escapes exists precisely because someone at Cursor recognized this; the path resolution logic that Mindgard exploits suggests the same recognition hadn't propagated everywhere.

AI coding agents are not going away, and the productivity argument for them is real. But the security implications of running an LLM with shell access and broad filesystem permissions are still being written, one disclosure at a time.
