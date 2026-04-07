---
title: "Two Models, One Keystroke"
date: 2026-04-07T06:22:42Z
draft: false
slug: ghost-pepper-local-pipeline
tags: [local-ai, speech, apple-silicon]
params:
  author: AI Beat Desk
  summary: >-
    Ghost Pepper v2.0.1 is a macOS hold-to-talk tool that quietly
    chains WhisperKit and a local Qwen 3.5 model to transcribe and
    clean up speech without any cloud call. It's a small app, but a
    clear signal of where on-device AI composition is heading.
---

[Ghost Pepper](https://github.com/matthartman/ghost-pepper) is a macOS app for dictation: hold Control, speak, release, and your words appear in whatever text field has focus. Nothing new about hold-to-talk. What's new is the pipeline underneath it.

Version 2.0.1 (released April 6) runs two AI models in sequence, both entirely on-device. The first is [WhisperKit](https://github.com/argmaxinc/WhisperKit)—Apple's port of OpenAI's Whisper—which handles speech recognition. The second is a Qwen 3.5 model in the 0.8B to 4B range, depending on what the user installs, which handles "smart cleanup": stripping filler words, correcting self-interruptions, fixing obvious transcription artifacts.

The Whisper step converts audio to text. The LLM step makes that text usable.

This is a mundane description of what is actually a fairly sharp design. The failure mode of raw ASR transcription is well-known to anyone who's used voice dictation seriously: you get "um" and "uh" and "actually no wait" and doubled words scattered through otherwise coherent sentences. Cleaning this up with rule-based systems is fragile. Using a small language model that understands context to silently rewrite the transcript before pasting it is genuinely the right approach.

What makes this interesting beyond the feature is the hardware prerequisite. Ghost Pepper requires Apple Silicon M1 or later. This is because running Whisper and a 4B-parameter Qwen model simultaneously in real time—with low enough latency that the delay after releasing the key doesn't break flow—is only practical on Apple Silicon's unified memory architecture. The same thing on an Intel Mac would be too slow to be useful. A year ago, this pipeline would have gone to the cloud. Now it runs locally, instantly, on consumer hardware.

The app has 337 upvotes on Hacker News this morning, which for a personal-project release of a niche macOS utility is substantial. Comments suggest it competes directly with [SuperWhisper](https://superwhisper.com/) but with the advantage of being open-source and free. The LLM cleanup step in particular seems to be landing as a genuine differentiator—the app's competitors mostly stop at raw transcription.

There's a broader pattern here. On-device AI is increasingly composed: you chain multiple small models together rather than calling one big model. Whisper handles audio-to-text because it's specifically trained for that and runs efficiently; a small general LLM handles the language cleanup because that's what it's good at. Each model does one thing well, they share a process boundary, and the result is more capable than either alone. The glue is just enough code to pass the transcript from one to the other.

Ghost Pepper is a small app solving a small problem. But it's a clean example of what consumer-grade on-device AI composition now looks like on Apple Silicon—and the answer is: fast enough to be invisible.
