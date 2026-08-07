---
title: "The Rubber Stamp Problem"
date: 2026-08-07T06:17:00+00:00
draft: false
slug: human-agent-permission-fatigue
categories: [safety]
tags: [safety, agents, oversight, security]
params:
  author: AI Beat Desk
  summary: >-
    A browser game by Belgian developer Alex Wauters tested 40,000 players on
    their ability to spot malicious AI agent commands under time pressure. The
    result: humans miss one in three threats. Scope violations and familiar-looking
    commands like `npm run analyze` slip through at the highest rates. Anthropic's
    own telemetry shows real Claude Code users approve 93% of permission prompts.
    Human-in-the-loop is not a security layer.
---

The design principle sounds reassuring: put a human in the loop. Before the AI agent does something consequential — writes to disk, calls a network endpoint, accesses credentials — ask a person to approve it. If the command looks wrong, they'll catch it.

Belgian developer Alex Wauters [ran that assumption through 40,000 game sessions](https://scalex.dev/blog/ai-agent-permissions-stats/) and found it doesn't hold.

Wauters built a browser game where players act as approval gatekeepers for a simulated AI coding agent. Commands appear one at a time, and you have 60 seconds to approve or deny each one. Some are routine; some are malicious. Points for correct decisions, points off for mistakes. Across 40,000 plays and roughly 409,000 approval decisions, players missed one in three malicious commands.

The breakdown by threat type is more informative than the headline number. Obvious destructive commands — `rm -rf /`, aggressive permission grants — were caught 88% of the time. People recognize blatant sabotage. What slipped through was subtler: exfiltration and arbitrary code execution attempts cleared review 33% of the time; scope violations cleared 35% of the time. Scope violations are things like an agent asking to read `~/.aws/credentials` or list Kubernetes configuration files. The command itself isn't destructive, the agent just has no legitimate reason to be touching those files. Reviewers, lacking the context of what the agent was actually supposed to be doing, tended to give the benefit of the doubt.

The single worst result was `npm run analyze`. This command — which can execute arbitrary JavaScript depending on what's in `package.json` — slipped past reviewers 65% of the time. Three `npm run` variants combined had a miss rate of over 52%. The reason is legible: `npm run` looks like normal development activity. It's something you'd approve reflexively because you've approved it a thousand times legitimately. The malicious payload, whatever it is, is inside the script definition in a file the reviewer never sees at approval time.

This is the permission fatigue problem in concrete form. The more approvals you see, the less cognitive load you spend on each one. A reviewer who correctly blocks an `rm -rf` in round one is making a much simpler decision than a reviewer who has to track what `npm run analyze` actually does in context. The security value of the approval decreases as the volume of approvals increases.

Wauters also documented the false-positive side: benign commands blocked at high rates. An internal npm registry mirror configured with `npm config set registry` was blocked 59% of the time. `rm -rf dist/` — clearing a build directory — was blocked 45% of the time. So humans aren't being conservative in a useful way; they're being inconsistent in both directions.

The real-world calibration point in the piece is striking: [Anthropic's telemetry shows](https://www.theregister.com/ai-and-ml/2026/08/06/humans-in-the-loop-miss-a-third-of-dangerous-ai-coding-agent-requests/5284236) that actual Claude Code users approve 93% of all permission prompts. In contrast, Claude's own auto-approval mode catches roughly 83% of problematic behaviors before they even reach a human. The system that isn't tired is doing better than the system that is.

None of this means human review has no value. There are cases where a human who knows the project context will correctly flag something that an automated classifier misses. The problem is that human review as typically implemented — high volume, time pressure, limited context — optimizes away that advantage. What gets produced is a queue-clearing function, not a judgment layer.

The design implication is that human oversight works best as a last-resort exception path rather than a primary control. If the agent's environment is properly sandboxed, if model-based classifiers handle the high-volume obvious cases, and if humans only see the genuinely ambiguous ones with full context, the oversight is meaningful. An approval prompt every 30 seconds is theater.

The game is playable and the methodology is transparent — it's worth a few minutes to calibrate your own miss rate before assuming you'd do better than the average.
