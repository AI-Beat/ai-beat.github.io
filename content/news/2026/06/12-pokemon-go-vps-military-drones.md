---
title: "The Thirty Billion Scans You Didn't Know You Made"
date: 2026-06-12T08:30:00+02:00
draft: false
slug: pokemon-go-vps-military-drones
categories: [safety-policy]
tags: [data, privacy, computer-vision, military]
params:
  author: AI Beat Desk
  summary: >-
    Dutch newspaper Trouw revealed that Niantic Spatial's Visual Positioning
    System — trained on 30 billion scans by Pokémon Go players since 2021 —
    has been integrated with Vantor's military drone navigation software for
    GPS-denied operations. Players consented to transferable data rights in
    optional in-game terms, but were never told of possible military use, and
    once data is baked into a model, tracing it back is essentially impossible.
---

When Pokémon Go players scanned their local PokéStops — those little camera surveys that let you see digital creatures overlaid on real streetscapes — they weren't thinking about drone navigation in GPS-contested airspace. That's not a criticism; there's no particular reason they should have been. But it turns out [that's where at least some of those scans ended up](https://dronexl.co/2026/06/09/pokemon-go-scans-niantic-vantor-military-drone-navigation/).

Dutch newspaper Trouw reported last week that Niantic Spatial — the AI spinoff that emerged from Niantic's technology assets after Scopely acquired the games business in 2025 — trained a Visual Positioning System on approximately 30 billion of those player-recorded video scans. The VPS does what GPS cannot when satellite signals are jammed or spoofed: it matches camera imagery against a preloaded 3D map to fix a device's location by sight. Two recognizable reference points a few pixels wide can be enough to establish position.

In December 2025, Niantic Spatial signed a partnership with Vantor (formerly Maxar Intelligence) to combine the ground-level VPS with Vantor's Raptor aerial navigation software. The goal: coordinated GPS-denied operations for autonomous drones, with applications in modern combat environments where GPS spoofing is routine.

## The consent problem isn't new, but this case is unusually sharp

Players who chose to participate in the PokéStop scanning feature agreed to supplementary terms that granted Niantic transferable licensing rights over the recorded data. The legal basis is sound, or at least standard. What's notable is that "transferable" includes transferring to a defense and intelligence firm for weapons system development — something the terms did not mention, and that most players presumably did not contemplate.

Vantor added a particular twist: when Trouw asked directly whether the deployed system uses Pokémon Go imagery, Vantor said it would not use game data. When asked whether the model *had been* trained on those scans in the past, Vantor declined to answer. This is not an unusual corporate non-denial, but it points to a genuine technical problem: once training data is absorbed into a model's weights, there is no reliable way for an outsider — or in many cases even an insider — to reconstruct exactly what the model learned from it. You can delete the source files. You cannot un-train the weights.

## Why this matters for AI training data generally

The Niantic case is a vivid instance of a broader structural issue: the supply chain for AI training data is routinely opaque about downstream use. Terms of service grant transferable rights; those rights get exercised in ways downstream users did not consider and were not told about; the resulting model's behavior is a permanent consequence that can't easily be audited.

The standard industry defense is that the data was collected within the terms users agreed to. That's legally defensible but it's not actually responsive to the concern, which is about informational asymmetry and reasonable expectation. Players who scanned streets for a mobile game had a reasonable expectation of the general category of use — augmented reality, location services, game improvement. Military drone navigation is a different category.

The deeper problem is architectural. For image diffusion models trained on scraped web content, this conversation has been ongoing for years. What's different here is that the data collection was *active* — users performed labor (recording videos) in exchange for an in-game reward — and the resulting military application is concrete and near-term, not hypothetical. That makes the gap between what users were told and what the data became unusually legible.

Niantic Spatial said in a statement that the model was trained on data from players who consented to their scans being used to improve AR, mapping, and navigation technology. That's accurate. It's also worth noting that navigation technology for GPS-denied military drones fits that description if you squint at it long enough.
