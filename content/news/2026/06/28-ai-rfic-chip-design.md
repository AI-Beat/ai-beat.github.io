---
title: "The Circuits AI Designs That No Human Would Have Drawn"
date: 2026-06-28T06:30:00Z
draft: false
slug: ai-rfic-chip-design
categories: [hardware]
tags: [hardware, chip-design, reinforcement-learning, rfic]
params:
  author: AI Beat Desk
  summary: >-
    Princeton's Kaushik Sengupta describes in IEEE Spectrum how reinforcement
    learning and electromagnetic emulation have crossed a threshold in radio
    frequency chip design: AI-generated circuits now routinely outperform
    human-designed ones, and the layouts look like QR codes — novel topologies
    that no human designer would produce or easily read.
---

Radio-frequency integrated circuit design is one of those fields where decades of
expert intuition have hardened into craft that resists codification. An RFIC
designer does not just apply a textbook formula; they develop, over years, a
feeling for how RF signals behave in silicon — what parasitics will matter, how
to lay out a circuit so the real-world electromagnetic environment cooperates with
the intended schematic, when to break convention. This knowledge is expensive to
acquire and almost impossible to transfer systematically. Which is why
[this week's IEEE Spectrum feature](https://spectrum.ieee.org/ai-radio-chip-design)
by Princeton's Kaushik Sengupta is worth reading carefully.

The system Sengupta's group built works in two stages. The first is a
reinforcement learning loop that explores circuit topologies without relying on
human-designed templates — it chooses architecture, device parameters, and layout
geometry autonomously. The second is an electromagnetic emulator: a convolutional
neural network trained to predict how a given circuit will behave
electromagnetically, replacing what would otherwise be a simulation step that
takes minutes to hours per design iteration. With the emulator in place, the RL
agent can evaluate thousands of candidate designs in the time a human designer
would spend on one conventional simulation.

The results from a 2023 experiment illustrate the scope of what this enables. The
system designed a 30–100 GHz broadband power amplifier — a wide-bandwidth,
high-frequency device where silicon-based designs have historically bumped against
hard limits — and achieved what the paper describes as the best combination of
bandwidth, output power, and efficiency then reported for a silicon power amplifier
at those frequencies. It also handled multiport circuits, where full
electromagnetic simulation had previously taken days or weeks per design. The AI
compressed that to minutes.

The more striking detail is what these circuits look like. AI-designed RFICs
apparently resemble QR codes: densely irregular, without the structured symmetry
that makes human-designed schematics readable. When an RL agent is not constrained
to human-legible topologies, it exploits geometrical degrees of freedom that
designers intuitively avoid — not because those geometries would not work, but
because circuits that look like random noise are difficult to reason about, verify,
or modify by hand. The AI is not designing schematics. It is discovering physical
configurations that satisfy the performance objectives, and those configurations
happen not to map onto anything in a textbook.

This is not entirely novel as a phenomenon. The same dynamic appeared in AlphaGo's
"alien" moves: the system produced valid, winning plays that professional players
could not have derived from existing theory. In RFIC design the implications are
more tangled, because a chip that works well but cannot be understood by the
engineers who would produce, validate, or iteratively improve it has real costs
downstream.

Sengupta's group has added diffusion models specifically to give designers control
over how "alien" the output looks. You can dial between fully novel topologies and
layouts constrained to resemble human-interpretable designs, trading some
performance for legibility. That lever is practically useful: a circuit that
requires weeks of expert analysis to validate has a hidden cost that does not show
up in the benchmark.

The honest limitation of the whole approach is data. RFIC design relies on
electromagnetic simulation outputs generated in-house and protected by NDA.
Unlike code repositories or scientific papers, there is no large public dataset
of RF circuit simulation results available for training general models. Sengupta
explicitly flags this: the field needs shared, contributed simulation datasets to
build foundational models with the kind of coverage that would generalize across
process nodes, frequency bands, and circuit types. This is not a problem that
more compute or better algorithms can solve on their own — it requires the
industry to decide that the collective benefit of shared training data outweighs
the competitive advantage of keeping it proprietary. For a field built on NDAs
and process-level trade secrets, that is a real cultural shift, not a technical
one.

What has been demonstrated already is that RL-based RFIC design does not merely
replicate human performance on easy cases. It consistently exceeds best-known
human results for the hardest class of silicon RF circuits, and it does so without
requiring a human to propose a starting topology. For an engineering domain where
design cycles have historically been measured in months and chips in hundreds of
millions of dollars, the case for AI-led design is getting harder to argue against
on merit alone — the remaining arguments are about validation, interpretability,
liability, and data access, which are real but have nothing to do with whether the
circuits work.
