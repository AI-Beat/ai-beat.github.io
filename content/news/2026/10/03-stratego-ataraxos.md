---
title: "The Last Board Game Falls"
date: 2026-10-03T08:11:44+02:00
draft: false
slug: stratego-ataraxos-superhuman
categories: [research]
tags: [reinforcement-learning, game-playing, imperfect-information, stratego]
params:
  author: AI Beat Desk
  summary: >-
    A six-person academic team built Ataraxos, the first superhuman Stratego player,
    for under $8,000 — roughly 1/500th of DeepMind's failed attempt. The key was
    pairing self-play RL with a decision-time generative model that tracks opponent
    beliefs, a design that transfers directly to any partially observable problem.
---

Chess fell in 1997. Go fell in 2016. Poker fell in 2019 — not perfectly, but well enough to beat professionals consistently. Stratego was supposed to be different.

The reason Stratego resisted so long is precisely what makes it hard: you never know where your opponent's pieces are. The game starts with each player arranging 40 pieces, face-down, across a 10×10 board. High-value pieces (the Marshal, the General) are hidden from view. Weak pieces might be sacrificed to probe what you're facing, or they might be the Marshal itself, disguised. Every reveal changes the entire probability landscape of where everything else is. The state space isn't just enormous — it's partially observable, and in ways that reward deliberate deception.

DeepMind tried. Their DeepNash system used Regularized Nash Dynamics and reportedly cost somewhere between $3 million and $4.5 million to train. It could beat most humans but never cracked the top echelon. The world's best players — the ones who've spent decades developing theories about piece placement and bluffing patterns — remained safe.

Until now. A [paper published in Nature on September 30](https://www.nature.com/articles/s41586-026-11036-y) by Samuel Sokota, Gabriele Farina, and colleagues from Carnegie Mellon, MIT, NYU, and Stanford introduces **Ataraxos**, which beat Pim Niemeijer — four-time world champion and the most decorated player in the game's organized history — 15 wins, 1 loss, 4 draws in an official 20-game series. It also went 38-2 against championship-level tournament players.

Training cost: under $8,000.

## What's actually different

The architecture has two components, and the combination is what matters.

The first is a **blueprint strategy** built through self-play reinforcement learning. The researchers found a key regularization technique that prevents the AI from converging on predictable early-game piece placements — the kind of detectable pattern that expert humans exploit ruthlessly. Getting this right is what killed prior approaches: without it, you either learn exploitable setups or you waste training cycles on strategies that can't generalize.

The second component is a **decision-time generative model** that, during actual gameplay, samples possible opponent board states given the moves and reveals observed so far. Rather than acting on a single best-guess belief about where the enemy Marshal is, Ataraxos generates a distribution of plausible configurations and plans against that uncertainty directly. This is why Farina calls it "the missing piece" — the blueprint gets you to expert level, but the generative belief tracking is what lets the system adapt to a specific opponent's revealed patterns in real time.

The 98%+ reduction in training compute compared to DeepNash comes mostly from the more efficient RL algorithm. The researchers describe the computational savings as one of their main contributions, not a side note.

## Why the cost matters more than the win

The $8,000 figure would've sounded absurd three years ago. But the more interesting question it raises is about the method's generality. Stratego is a clean instance of a class of problems — partially observable, adversarial, long-horizon, with deliberate opponent deception — that appears constantly outside games. Intelligence analysis, cybersecurity (specifically adversarial red-teaming), negotiation, multi-agent coordination under uncertainty: all share this structure.

The prior art on imperfect-information game-solving was either compute-intensive (CFR variants) or specialized (poker-specific techniques). What Sokota et al. show is that general self-play RL plus a generative opponent model is enough to reach superhuman performance in a game this hard, at a cost accessible to academic groups. That's the interesting result, more than the win streak.

The paper also shows the approach working on Hanabi and Dou dizhu — different games, same fundamental structure. The generalization isn't accidental; it's the point.

Stratego held out longer than most expected. It was a reasonable bet that hidden information at this scale would require either massive compute or game-specific tricks. It turned out to require neither.
