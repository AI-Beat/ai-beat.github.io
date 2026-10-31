---
title: "Trojanized in the Marketplace"
date: 2026-10-31T06:09:30+00:00
draft: false
slug: claude-code-vsx-backdoor
categories: [security]
tags: [security, tools, supply-chain]
params:
  author: AI Beat Desk
  summary: >-
    Phorion, a macOS endpoint security vendor, published an advisory on October
    30 describing a backdoor in versions 2.1.312 and 2.1.313 of the Claude Code
    VS Code extension distributed through the Cursor Marketplace. The payload
    installs an attacker SSH key, opens a reverse tunnel, and persists as a fake
    Apple system service. Cursor has removed both versions.
---

Phorion, a macOS endpoint security company, [published an advisory](https://phorion.io/blog/offical-claude-code-vsx-compromise/) on October 30 describing a backdoor in versions 2.1.312 and 2.1.313 of the Claude Code for VS Code extension as distributed through the Cursor Marketplace. Cursor has removed both versions. The advisory is worth reading in detail because the attack surface — developer tools consumed via extension marketplaces — is one the industry has not figured out how to secure at scale.

The payload is prepended to the extension bundle and runs when the extension loads, before any user interaction. On macOS it does several things: it appends an attacker-controlled SSH public key to `~/.ssh/authorized_keys`, attempts to enable Remote Login via `systemsetup`, opens a persistent reverse SSH tunnel to an external server (`134.209.82.225:22`), and installs a LaunchAgent for persistence disguised as `com.apple.sysmond` — a name chosen to blend in with legitimate Apple system monitor services.

There is a bug in the macOS path. The script checks whether Remote Login is already enabled using `systemsetup -getremotelogin`, which requires admin privileges; without them, the command errors rather than returning a status string. Since the script checks the output for the literal word "Off", an error response causes it to skip the `systemsetup -f -setremotelogin on` call. The net effect is that on most Macs where SSH wasn't already running, the inbound access path doesn't get opened — but the attacker's public key and the LaunchAgent persistence do still get installed. Phorion notes this shouldn't be treated as a defence.

The broader pattern here is not new, but the targeting is. AI coding assistant extensions are attractive targets for the same reasons that credential-harvesting extensions have always been attractive — developers are logged into everything, have push access to codebases, and often run with high local privileges — but there are additional reasons specific to AI tools. Claude Code reads and writes the filesystem on the developer's behalf, and a compromised extension running inside the same VS Code process as a heavily privileged AI agent creates a particularly clean path to sensitive data. The earlier [Nx Console incident](https://labs.cloudsecurityalliance.org/research/csa-research-note-vscode-extension-supply-chain-breach-20260/) from May 2026 also targeted Claude Code configuration files specifically, which suggests the tooling has become a named target rather than an accidental victim.

The Cursor Marketplace draws its extensions from Open VSX, the open-source registry that VS Code forks use in place of Microsoft's proprietary Marketplace. Open VSX has a different publication process from the VS Code Marketplace: there is no bytecode signing, the review process is lighter, and the ranking algorithm can be influenced by review manipulation. Getting a trojanized version of a well-known extension published is harder on Microsoft's Marketplace than it used to be, which means supply-chain pressure has shifted toward the alternatives.

None of this is unique to AI tools. The VS Code extension ecosystem has had supply-chain incidents involving cryptominers, credential stealers, and remote access trojans for years. What changes when AI developer tools become the target is the blast radius: the extension is running inside a process that may itself be running commands, reading files, and making network requests as a normal part of its job. Distinguishing malicious traffic from legitimate AI agent activity is harder than distinguishing malicious traffic from a syntax highlighter.

Affected users should check `~/.ssh/authorized_keys` for unknown keys, look for a LaunchAgent named `com.apple.sysmond` in `~/Library/LaunchAgents/`, and review outbound connections to `134.209.82.225`. Phorion's advisory includes full file hashes and indicators of compromise.
