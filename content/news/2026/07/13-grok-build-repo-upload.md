---
title: "What Grok Build Uploads"
date: 2026-07-13T06:12:26+00:00
draft: false
slug: grok-build-repo-upload
categories: [security]
tags: [security, xai, grok, coding-agents, privacy, tooling]
params:
  author: AI Beat Desk
  summary: >-
    A wire-level analysis of Grok Build CLI 0.2.93 found it uploads the entire
    workspace as a git bundle to Google Cloud Storage — about 5.1 GiB from a
    12 GB repo, including files the agent never read and unredacted .env
    credentials. The model itself received 192 KB. The "Improve the model"
    toggle does not stop the upload.
---

A researcher who goes by cereblab published [a wire-level analysis of xAI's Grok Build CLI](https://gist.github.com/cereblab/dc9a40bc26120f4540e4e09b75ffb547) today, and the numbers are unusual enough to be worth understanding carefully before you run the tool on a production codebase.

The methodology: a logging proxy spliced between Grok Build CLI (version 0.2.93) and the model endpoint, capturing exact JSON payloads, HTTP transactions, and upload sizes. Three distinct transmission channels were identified.

The first is what you'd expect: the model-turn channel. Files Grok reads get sent as context, in this case about 192 KB including `.env` files with API keys and database passwords in the clear.

The second is less expected. During each session, the CLI sends the entire workspace to a Google Cloud Storage bucket — `gs://grok-code-session-traces` — as a git bundle. On the test repository (12 GB), the captured upload was 5.10 GiB — transmitted in 73 chunks across 83 requests, all returning HTTP 200 — with the researcher noting the capture was truncated mid-stream, so the actual transfer may be larger. The upload is not scoped to files Grok touched. When the researcher added files with unique embedded markers that the agent never opened, those markers arrived intact in the recovered git bundle. Everything in the workspace goes, not just what the agent read.

The third channel is telemetry: Mixpanel tracking plus xAI event logging.

The ratio that anchors the story: the model received 192 KB of context; storage received 5.1 GiB. That is roughly a 27,800× disparity between what the agent processed and what left the machine.

The researcher is explicit that this "does not prove xAI trains on this data" — that's a policy question separate from the technical finding that the data is transmitted and stored. What the analysis establishes is that transmission happens, that it includes a full snapshot of your repository regardless of agent activity, and that credentials travel unredacted.

The privacy toggle does not help here. Disabling "Improve the model" in the Grok Build settings changes the training consent flag — but when the researcher toggled it off, the `/v1/settings` API response still returned `"trace_upload_enabled": true` and `"upload_enabled": true`. The upload appears to be a separate mechanism, presumably for debugging and session reconstruction, and it runs regardless of training consent.

Most coding agents send data to cloud services. That's expected — the model runs remotely, context has to travel. What is different here is the *scope*. The standard expectation is that what you send matches roughly what the agent sees: the files it reads, the directory tree it explores, the diffs it generates. A multi-gigabyte upload of the full repository, including paths the agent never touched, is a different behavior, and it's not disclosed in the Grok Build documentation.

For solo developers working on public or non-sensitive code this may not matter much. For anyone running Grok Build against a repository containing database credentials, API keys, internal infrastructure code, or anything covered by a data residency policy, the gap between what was disclosed and what was transmitted is worth thinking about before the next session.

xAI has not responded publicly. The full analysis, including packet captures and hash chain for the audit trail, is in [the original gist](https://gist.github.com/cereblab/dc9a40bc26120f4540e4e09b75ffb547).
