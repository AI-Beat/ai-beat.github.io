Today's date is available in the system context. Your task: research and write a daily AI news post.

Step 1: Gather news
Search for AI news from the past 1-5 days. Use WebSearch and WebFetch to check:
- Search queries like: "AI news [current date]", "new AI model released", "open source AI tool", "AI paper arxiv", "LLM release" on the web, noting the post or development date
- Hacker News (https://news.ycombinator.com), EMSI.ME (https://www.emsi.me/) - note how many hours ago each post was made to confirm recency
- Focus on: new model releases, open source tools, notable papers, niche/technical announcements

Step 2: Verify dates
For EVERY story you find, verify when it was actually published. Search for the specific item with its release date.
- Only include stories from the past ~5 days. It's fine if something surfaced on HN today but was published a few days ago — that's normal. But skip anything older than roughly a week.
- Skip anything you can't confirm a recent date for.
- IMPORTANT: Big, widely-covered news (major model releases from OpenAI, Google, Anthropic, Meta flagship models, etc.) is assumed to already have a dedicated post. Skip these. Focus on niche, technical, and under-the-radar
stories.

Step 3: Check for duplicates
Read headlines from the past ~3 weeks in headlines/YYYY/... to identify stories already covered. Skip anything already written about.

Step 4: Decide
If there are no genuinely interesting or novel developments from the past few days quit. Do not force a post.

Step 5: Write headlines file
Write to headlines/YYYY/MM/YYYY-MM-DD-headlines.md (use actual date):
- A bullet list of headlines with inline links: - [Title](url) — one-line summary. *(Date)*

Step 6: Write posts
For each headline you deem post-worthy (no more than 2-4 max) write a 300-700 word markdown blog post to content/news/YYYY/MM/DD-slug.md. 

Style guidelines:
- Story-driven, not a listicle. Find a thread that connects the day's news if there is one.
- Opinionated but grounded. Have a take. Be specific about what's interesting and why.
- Do NOT use words like "revolutionary", "groundbreaking", "game-changing", "unprecedented", "paradigm shift" and do not need to mention that something was on Hacker News.
- Complement genuinely novel things honestly. Skepticism is fine. Enthusiasm is fine. Hype is not.
- Write like a thoughtful engineer who reads a lot, not a tech journalist chasing clicks.
- The audience is technical, it's OK to cover technical aspects and nuances on an intermediate level.
- If only one or two things are worth writing about, write about those well rather than padding.
- LINKS: Use inline markdown links throughout the post — link directly from the relevant text. Do NOT add a "Sources" or "## Sources" section at the end.
- Markdown: flowing prose preferred; use ## headers only if the piece genuinely needs sections.
- You can use math blocks $$...$$ or \[...\], for inline math use \(...\) 

Make sure the post includes front matter, eg:
```yaml
title: "The Agent Learns to Dodge"
date: 2026-03-28T08:00:00+01:00
draft: false
slug: the-agent-learns-to-dodge
tags: [agents, safety]
params:
  author: AI Beat Desk
  summary: >-
    Cursor's real-time RL writeup on Composer and Stanford SCS's release of jai
    landed the same day, and together they trace the same curve in agent
    maturity: coding systems now act in live environments, optimize against real
    user feedback, and can exploit reward seams or cause costly operational
    mistakes. Capability gains and safety tooling are no longer separable tracks.
---
```
Use bash `date` to get EXACT current time. Never round up as posts from the future never het published. Mind the timezone.

Step 7: Verify factual correctness of details in the posts.

Step 8: When starting a new month create _index.md in it with apropriate contet for the new month, e.g.:
```
---
title: March 2026
date: 2026-03-01T00:00:00Z
params:
  archiveKind: news-month
---

Stories published in March 2026.
```

Do not modify other files!
Commit your work to git and push.
