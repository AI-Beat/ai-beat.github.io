---
title: "Bus Traffic as Source Material"
date: 2026-10-16T06:10:00+00:00
draft: false
slug: ai-hardware-reverse-engineering-linux-drivers
categories: [tools]
tags: [linux, reverse-engineering, hardware, drivers]
params:
  author: AI Beat Desk
  summary: >-
    Christian Schaller used Claude Code and a USB traffic capture harness to
    write a SANE backend for a Plustek film scanner, a Rust app for scanning
    negatives, and kernel plus libcamera fixes for a ThinkPad MIPI camera —
    a concrete account of AI taking over the tedious comparison work in
    hardware reverse engineering.
---

Hardware reverse engineering has always been a patience game. You capture bus traffic, stare at hex sequences, compare traces between a vendor's working Windows driver and your own incomplete Linux driver, and try to figure out which register write is causing the scanner to skip the lamp initialization sequence. It's work where the answer is often visible in the trace, if you look carefully enough — but "carefully enough" might mean hours of manual diff.

Christian Schaller, a GNOME developer at Red Hat, [wrote up this week](https://blogs.gnome.org/uraeus/2026/10/15/aiandreverseengineerhardware/) how he automated the comparison layer using Claude Code and a harness he calls hw-harness. The setup is straightforward: the vendor's Windows driver runs inside a KVM virtual machine with the real device passed through via USB. The Linux host captures USB traffic with usbmon — the guest can't see this capture. Claude Code then compares the reference trace from the vendor driver against traces from Schaller's in-progress Linux driver and surfaces the differences. For devices with no capturable bus, like an internal MIPI camera, the AI worked from Windows-side artifacts instead: INF files, registry entries, ACAP tables, device properties.

Three concrete projects came out of this work.

The first is a SANE backend for the Plustek OpticFilm 8200i, a film scanner built on a Genesys GL128 chip (USB 07b3:1825) that had no prior Linux support. The backend covers color, grayscale, and infrared modes, resolutions from 300 to 7200 dpi, and infrared dust removal. Performance landed well: 34.6 seconds at 3600 dpi, compared to 54.4 seconds for the upstream SANE genesys backend (which added GL128 support in September 2026, during the project — some competition there).

The second is Negative Vibes, a Rust GNOME application for scanning color and black-and-white negatives and slides. It keeps scans as 16-bit linear data, applies non-destructive edits, handles IT8 calibration through ArgyllCMS, and exports to TIFF, JPEG, PNG, and DNG. That it became a full standalone application — not just a driver — is a notable outcome. The AI apparently handled enough of the protocol analysis that Schaller had headroom to think about the application layer.

The third is a series of fixes for the ThinkPad X1 Carbon Gen 14's MIPI camera, which uses a Sony IMX471 sensor behind an Intel IPU7. The kernel side had a crop-selection bug, a wrong test-pattern register width, and a low analogue gain cap. The libcamera side had missing lens shading correction and white-balance statistics summed at the wrong bit depth. After the fixes, frame brightness went from 31.1 to 90.9, though blue remains about 17% short in low light — some things require more work.

Schaller is clear about the division of labor. The hardware remained essential throughout. No trace analysis tells you whether the scanner lamp actually moved, or whether the image coming out of the MIPI pipeline actually looks right. As he puts it: "Real hardware is the only judge." The AI's contribution was in the analysis layer — comparing traces, reading INF files, cross-referencing ACPI tables against register documentation — the parts where the information is already present and the task is pattern matching rather than invention.

That's a useful distinction. The bottleneck in driver writing isn't usually finding the source code for how USB control transfers work; it's the tedious trace comparison that tells you which undocumented register your driver is missing. Shifting that part to an AI changes what a solo developer can actually accomplish. The understanding still needs a human; the diff does not.

