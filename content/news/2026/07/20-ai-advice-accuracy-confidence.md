---
title: "Confident and Wrong"
date: 2026-07-20T06:09:49+00:00
draft: false
slug: ai-advice-accuracy-confidence-gap
categories: [research]
tags: [safety, research, human-ai-interaction, cognition]
params:
  author: AI Beat Desk
  summary: >-
    A controlled study from three European universities finds that AI assistance
    collapses human accuracy from 27% to 9% on questions AI gets wrong, while
    confidence nearly triples to 76% and willingness to say "I don't know"
    falls from 44% to 3%. The effect persists even with financial incentives to
    do better.
---

A [new study](https://thenextweb.com/news/ai-advice-suppresses-critical-thinking-wrong-answers-study) from researchers at the University of Milano-Bicocca, École Normale Supérieure, and Sapienza University of Rome, published yesterday, tests a specific and uncomfortable question: what happens to human judgment when AI assistance is available but confidently wrong?

The setup was designed to avoid a confound that plagues most AI-cognition research. If you test people on questions where AI excels — coding, factual recall, summarization — and find that they defer to its answers, you can't tell whether they're being rational (the AI is probably right) or uncritical (they've outsourced thinking). So the researchers deliberately chose questions where AI models typically fail: visual and contextual details from films. They then gave participants access to Claude 3.5 Flash, which frequently gave wrong answers.

The numbers are stark. Without AI access, participants answered correctly 27% of the time and were willing to say "I don't know" 44% of the time — not great on accuracy, but showing appropriate epistemic humility. With AI access, accuracy dropped to 9% (roughly one third of baseline), confidence rose from 30% to 76%, and admissions of ignorance collapsed to 3%.

The researchers tried adding financial incentives to improve this picture. It helped marginally — admission of ignorance rose to 8%, accuracy to 16% — but didn't come close to recovering baseline performance. Knowing that being wrong costs you something didn't restore the cognitive habit of recognizing when you don't know.

The core finding isn't really about accuracy on trivia questions. It's about metacognition. One of the most useful things a person can do is recognize the edge of their own knowledge and say so. That habit appears to be suppressed simply by having an AI present that sounds confident. The AI's confidence gets imported into the human's confidence, even when the AI is wrong, and even when the stakes are real enough to matter.

This connects to something [observed in enterprise AI deployments](https://ai-beat.github.io/news/2026/07/ai-mania-decision-making/) — executives gaming metrics, organizations losing the capacity to register that projects are failing. What the study isolates is the individual-level mechanism: AI availability doesn't just change what answers you give, it changes whether you think to question them.

The study used Claude 3.5 Flash rather than a frontier model, which is worth noting. A stronger model would have gotten more of the test questions right, which would have changed the experimental setup rather than the underlying dynamic. The research interest isn't "Claude 3.5 Flash is unreliable" — it's that the cognitive pattern (import confidence, suppress doubt) seems to operate independently of the AI's actual accuracy. You'd presumably see the same dynamic with a better model on the questions it gets wrong, since you still can't easily tell from the model's behavior when those are.

The researchers frame this as a concern for developing minds specifically, but the data doesn't really support limiting the concern to students. Adults with financial incentives showed the same pattern at smaller magnitude. The habit of epistemic humility — pausing to ask "am I sure, or am I just repeating what I heard?" — doesn't seem to survive the presence of a confident AI interlocutor.

There isn't an obvious design fix here. Injecting uncertainty markers into AI outputs ("I'm not sure about this") doesn't hold up well empirically — models that hedge more tend to be trusted less overall, which creates different problems. What the study suggests is that the problem may be more behavioral than informational: not that people need better calibration signals from the AI, but that they need the habit of checking to survive contact with a tool that's designed to sound confident.
