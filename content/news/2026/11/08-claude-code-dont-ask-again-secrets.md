---
title: "The Approval That Persists"
date: 2026-11-08T07:00:00+01:00
draft: false
slug: claude-code-dont-ask-again-secrets
categories: [security]
tags: [claude-code, secrets, github, agentic-tools, security]
params:
  author: AI Beat Desk
  summary: >-
    Apiiro scanned 104,185 public copies of Claude Code's local settings file and found
    real credentials in 1,122 of them — database passwords, AWS keys, SSH logins. The
    root cause is a UX pattern that stores approved commands verbatim in a project file,
    combined with a gitignore that only lives on the developer's machine.
---

There's a certain class of security failure that isn't a bug in the traditional sense — no CVE, no malicious actor, no memory corruption. It's a confluence of reasonable-seeming design choices that together produce a bad outcome at scale. The [Claude Code "don't ask again" secret leak](https://runtimewire.com/article/claude-code-dont-ask-again-secrets-github-apiiro) reported by Apiiro on November 5 is one of those.

The mechanism is worth spelling out precisely, because the individual pieces each seem defensible. When Claude Code needs to run a shell command, it asks the developer for permission. For repeated commands, there's a "Yes, and don't ask again" option — a standard UX affordance for reducing prompt fatigue. The tool stores the approved command in `.claude/settings.local.json`. If the command contained a secret — a database URL with an embedded password, say, or a curl call with an API key — that secret is now sitting in that file verbatim.

The file is meant to stay local. Claude Code creates a gitignore rule for it, so on the developer's machine it never ends up in version control. The problem: that gitignore rule only exists on the developer's machine. When the project is pushed to a new repo, forked, deployed via CI, or committed by a colleague who doesn't have the same local configuration, the protection is gone. The file gets committed. The secret goes with it.

Apiiro scanned 104,185 public copies of `.claude/settings.local.json` on GitHub. [1,122 of them contained at least one real credential](https://thehackernews.com/2026/11/claude-codes-dont-ask-again-file.html). Across 1,994 repositories, counting settings files in git history and agent transcripts in addition to current files, the total rises to 2,539 secrets. The breakdown is unglamorous but concrete: 379 database passwords, 43 SSH credentials for public servers, 42 tokens for GitHub, GitLab, and npm, 23 AWS keys.

The practical severity depends on whether the keys still work, which Apiiro's research didn't fully test — but at least one exposed credential was confirmed live by a company that recognized their own key in the scan.

What makes this worth thinking about beyond the immediate "rotate your credentials" response is that it illustrates a tension inherent to agentic coding tools. These tools need persistent context — approved commands, project preferences, remembered choices — and that context naturally lives in files alongside the code. The same practices that work fine for personal preferences (font size, tab width) are genuinely dangerous when applied to credentials. A config file for personal settings can safely be gitignored once and forgotten; a config file that might contain secrets needs to either be gitignored in the project's shared `.gitignore` or stripped of secrets before it can ever be committed. The local-only gitignore pattern fails exactly when code moves between machines or hands.

Anthropic hasn't yet published a public response. The fix isn't complicated in principle — redact argument values from stored approval rules, or add the path to the project's shared gitignore on initialization. But until that ships, the practical advice is defensive: add `.claude/settings.local.json` to your `.gitignore` explicitly, check git history for prior commits containing the file, and treat any credential ever passed to Claude Code as potentially exposed if it was part of an approved command. Deleting the file from HEAD isn't enough if the credential has been in git history — the right step is to rotate the credential.

The lesson that travels: when a tool stores anything on behalf of a developer, it should be explicit about what it's storing, where, and whether that location is safe to commit. "Local only" is an assumption that breaks every time code is shared — which, for most projects, is the entire point.
