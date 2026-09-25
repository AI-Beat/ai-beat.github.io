---
title: "Four TPUs at Altitude"
date: 2026-09-25T06:13:11+00:00
draft: false
slug: suncatcher-tpus-orbit
categories: [infrastructure]
tags: [hardware, compute, space, google, tpu]
params:
  author: AI Beat Desk
  summary: >-
    Google's Project Suncatcher scheduled its first hardware test for October 1 on SpaceX's Transporter-18 rideshare: four Trillium TPUs going to low Earth orbit to test radiation tolerance, thermal cycling, and launch forces. The companion research paper describes the full engineering stack — free-space optical links at 1.6 Tbps, close-formation station-keeping with Hill-Clohessy-Wiltshire equations, and economics that only close if launch costs fall below $200/kg by the mid-2030s.
---

The bet at the center of [Project Suncatcher](https://blog.google/innovation-and-ai/models-and-research/google-research/google-project-suncatcher-facts/) is that the hard constraint on AI compute isn't silicon — it's Earth. Data centers compete for power grid access, cooling water, and physical land. Solar panels on the ground convert sunlight inefficiently, behind atmospheric losses and day/night cycling. In sun-synchronous low Earth orbit you get near-constant sunlight above the atmosphere, and 3K passive radiative cooling instead of cooling towers. The physics of energy collection in space are genuinely better.

[Google announced September 24](https://gizmodo.com/googles-project-suncatcher-is-sending-ai-chips-into-space-next-week-2000816985) that it will test whether this is more than a physics argument. Four Trillium TPUs are manifested on SpaceX's Transporter-18 rideshare launch scheduled for October 1, in partnership with Planet Labs. The mission is hardware validation: can AI chips survive launch forces, radiation, and thermal cycling in orbit? This is roughly a year ahead of the informal timeline from when the project was first announced.

The radiation question is specific. The team tested Trillium TPUs in 67 MeV proton beams — the high-energy particles that dominate the LEO radiation environment — and the memory subsystems tolerated nearly three times the expected five-year mission dose. That's a real engineering data point.

The [research paper published alongside the announcement](https://research.google/blog/exploring-a-space-based-scalable-ai-infrastructure-system-design/) describes what a production constellation would actually require. The central problem is inter-satellite communication at scale: a useful network needs tens of terabits per second between nodes. Google's approach uses dense wavelength-division multiplexing with spatial multiplexing over free-space optical links — bench testing demonstrated 1.6 Tbps total throughput. To make the power budget work for optical links, satellites need to fly close together: less than 1 km apart, with 100–200m as the target operating distance.

Keeping satellites that close requires careful orbital mechanics modeling. The team used [Hill-Clohessy-Wiltshire equations](https://en.wikipedia.org/wiki/Clohessy%E2%80%93Wiltshire_equations) with differentiable JAX-based refinements to predict how a constellation in 650km sun-synchronous orbit responds to gravitational perturbations. Analysis shows 100–200m spacing is stable with only modest station-keeping maneuvers — the thruster budget is manageable.

The economics case is honest about its conditions. At current launch costs, space-based compute doesn't make financial sense. The paper models a scenario where costs fall below $200/kg by the mid-2030s, at which point the comparison with terrestrial data centers could close on a per-kilowatt-per-year basis. That's a defensible extrapolation given the SpaceX and Rocket Lab cost trajectories, but it's still a decade-plus bet contingent on things going well in launch markets.

The more grounded near-term question is what happens on October 1. If the hardware survives and behaves within spec, Google has a baseline for a larger prototype mission scheduled for early 2027, which aims to validate distributed ML workloads running between two satellites. The gap between "chips survived the ride up" and "we ran inference across a satellite constellation" is substantial, but the October test is the necessary first step.

Putting AI compute in orbit isn't an obvious solution to the terrestrial infrastructure problem. It's a long-horizon bet that the constraints only get worse on the ground, and that building a different infrastructure architecture now is worth the engineering cost. How October goes will say something about whether the hardware half of that bet holds.
