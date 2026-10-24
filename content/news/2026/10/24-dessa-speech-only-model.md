---
title: "The Conversation Without Words"
date: 2026-10-24T06:11:38+00:00
draft: false
slug: dessa-speech-only-conversation-model
categories: [research]
tags: [speech, audio, conversation, multimodal]
params:
  author: AI Beat Desk
  summary: >-
    Block's Dessa is a full-duplex conversation model trained from scratch on
    human speech alone — no text, no language model backbone, no TTS pipeline.
    It's either a genuine new research direction or a very polished demo, and
    right now we can't quite tell which. Either way the framing is interesting.
---

There's a peculiar thing about how virtually every "voice AI" system works today: the audio is almost incidental. You speak, a transcription model turns that into text, an LLM reasons over the text, a TTS system converts the reply back into sound. The voice is a shell around a fundamentally text-native system. What you hear isn't a system that understands speech — it's a text model wearing a voice costume.

[Dessa](https://block.xyz/dessa), released by Block's AI research team this week, tries something different. It is trained only on human conversation audio. No text labels, no pretrained language model backbone, no intermediate representations. Audio goes in; audio comes out. The model has to learn what language *is* from the sound of people talking to each other, the same way a child does — through prosody, rhythm, pauses, laughter, correction, and the full texture of spoken exchange.

The practical consequence is that Dessa can do things a cascade pipeline structurally cannot. It's full-duplex: it listens and speaks at the same time, which means it can interject naturally, trail off, say "yeah" as a backchannel signal that it's tracking what you're saying, and pick up on whether you're being sarcastic from the way you said it rather than from parsing the sentence. None of that information survives transcription. Once you flatten speech to text, the timing is gone, the hesitations are gone, the emotional coloring is gone. A text model working on the transcript is flying blind on the paralinguistic layer.

Whether Dessa actually *understands* any of this in a meaningful sense is a harder question, and Block's page doesn't try to answer it rigorously. There are no benchmark numbers, only audio demos of conversations — the kind of evidence that makes anyone technically careful a bit nervous. The claim that Dessa "learns language from far fewer words than large language models" is interesting but unverified; it would follow naturally from learning from audio (a spoken corpus is much smaller than a scraped-web text corpus), but we don't know if the model generalizes in the ways that matter, or only in the ways that demo well.

The infrastructure claims are notable: the model was trained on a single NVIDIA GB200 NVL72, and inference runs on a MacBook Pro M4 Max or better. Those are genuinely encouraging numbers for a model trained from scratch. The team at Block has been working on deep learning since 2012 and generative speech models for over eight years, so this isn't a weekend project — but "we can build it" and "we've built something that works" are different thresholds.

What makes this worth watching is the research direction. The argument that text is a lossy intermediate representation is correct. The argument that a model trained directly on speech would understand conversation better is plausible. The question is whether current architectures and training approaches can actually exploit that structure, or whether text-native models are powerful enough that the lossy encoding doesn't matter much in practice. Dessa is a bet that it does matter, made with actual weight files rather than a blog post.

Access right now is limited to a small number of AI safety researchers. More recordings are promised on X. Open weights aren't mentioned. Whether Block intends this as a serious research release or a proof-of-concept is not entirely clear from the page — but the demo exists and the idea is genuine, and that's more than most "voice AI" announcements in the past two years have offered.
