---
title: "The Dog Still Won't Fetch, But the Gap Is Closing Fast"
date: 2026-06-21T06:05:41+00:00
draft: false
slug: project-fetch-phase-two
categories: [research]
tags: [anthropic, robotics, agents, claude, embodied-ai]
params:
  author: AI Beat Desk
  summary: >-
    Anthropic's Phase Two of Project Fetch has Claude Opus 4.7 completing a four-task
    robotic quadruped challenge nearly 19× faster than a human team with AI assistance
    and generating a tenth of the code — through no robotics-specific training. The
    robot still can't autonomously retrieve the beach ball. That combination of
    dramatic capability transfer and stubborn physical limits tells you something
    interesting about where general AI scaling is and isn't working.
---

Last August, Anthropic ran a small experiment: give non-expert employees a robotic quadruped and see how much Claude could help them get it to do useful things. The [result](https://www.anthropic.com/research/project-fetch-robot-dog) was interesting but messy — humans working with AI could do more, but the feedback loops were slow and the code got unwieldy.

[Phase Two](https://www.anthropic.com/research/project-fetch-phase-two) cuts out the humans almost entirely.

For the follow-up, Anthropic gave Claude Opus 4.7 autonomous access to the same off-the-shelf robot dog — connecting to the lidar and cameras, writing control programs, detecting objects, navigating the environment — and let it run. Across four tasks that all teams completed, Opus 4.7 finished in 9 minutes and 35 seconds. The fastest human team (aided by Opus 4.1) took 181 minutes. The team working without an AI model took 361 minutes.

That's 18.9× faster than the assisted human team and 37.7× faster than the unassisted one.

The code efficiency gap is just as striking. Team Claude wrote 10,309 lines of code over the experiment. Team Claude-less wrote 1,136. Opus 4.7 wrote 1,045 — less than the unaided humans, while outperforming them on every completed task, and with much of it working on the first try.

Worth noting: Opus 4.7 was not trained on robotics tasks. It wasn't fine-tuned on robot control code, sensor integration, or quadruped kinematics. The performance improvement came from the model getting better at reasoning in general, and that general reasoning transferred cleanly to a physical problem. This is the core argument for scaling general AI rather than training narrow domain models for every application — and Phase Two is a clean demonstration of it working in practice.

But the task is called Project *Fetch* for a reason. The point was always to get the robot to autonomously retrieve a beach ball, and Opus 4.7 still can't do it.

The failure mode is precise: the model can navigate to the ball's approximate location and position the robot, but autonomous retrieval requires closed-loop control — real-time visual feedback that continuously adjusts the robot's movements as it approaches, grasps, and maneuvers the ball. The model's efforts at this were "poorly controlled and not successful." Anthropic notes that human researchers with actual robotics expertise could complete the fetch; the researchers believe current models *could* eventually manage it with more time and better scaffolding, but they haven't gotten there yet.

This distinction matters. The tasks Opus 4.7 smoked humans on were fundamentally open-loop: write code to connect to the sensor, write a detection routine, navigate to waypoint A, then waypoint B. These involve decomposing a task into a sequence of steps, writing correct code for each step, and debugging when something doesn't work — exactly what current LLMs are good at. Closed-loop control is different: it requires integrating continuous sensor streams and making real-time corrections with low latency and high precision, which doesn't translate naturally to the token-by-token reasoning these models do.

What's interesting here is that the two failure modes often cited for LLMs in physical settings — "they can't generate correct code for real systems" and "they can't do real-time physical control" — are now very much separable. The code generation problem appears largely solved for this class of task. The real-time control problem is still real.

Whether that second problem gets solved through better prompting, tighter agent scaffolding, specialized robotics fine-tuning, or just more scaling is unclear. The Phase Two result suggests general scaling gets you far, but not to the finish line — at least not yet.

The robot dog remains ball-less. The distance to the finish line is measurably shorter.
