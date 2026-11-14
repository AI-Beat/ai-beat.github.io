---
title: "600 Experiments in a World Model"
date: 2026-11-14T06:17:26+00:00
draft: false
slug: jepa-planning-600-experiments
categories: [research]
tags: [world-models, JEPA, robotics, planning, evaluation]
params:
  author: AI Beat Desk
  summary: >-
    Haddy Al-Chaer ran 600 experiments on LeWM, a JEPA world model for the
    Push-T robot task, testing every variable that seemed like it should matter.
    The main finding: the predictor quality is nearly irrelevant. Planning method
    and goal specification dominate. And the standard benchmark hides most of
    the difficulty.
---

JEPA world models have been building momentum as a theoretical alternative to next-token prediction — the idea being that a model predicting abstract representations of future states, rather than raw pixels or tokens, should generalize better and require less data. The theory is appealing. The empirical picture is messier, and [Haddy Al-Chaer](https://haddyalchaer.com/jepa) has done the unglamorous work of finding out exactly why.

The write-up describes 600 experiments on [LeWM](https://learnablerobotics.github.io/lewm/), a JEPA world model trained on the Push-T robot pushing task. The experiments test four planning methods, several goal-specification approaches, and a memory-graph variant — none of which retrain the model. The question isn't whether JEPA works as a learning method; it's whether, once you have the trained predictor, the planning layer you build on top of it is what determines performance.

## The predictor barely matters

The most striking result: on long-horizon tasks with expert-supplied subgoals, LeWM's learned predictor succeeded 78% of the time. Using the simulator's exact physics in place of the predictor: 77%. The predictor is not the bottleneck.

This is worth sitting with. The usual frame for evaluating world models focuses heavily on prediction accuracy — how well does the model generalize out of distribution, what's the rollout error at \(t+k\) steps? Those are real questions, but the results here suggest that at the planning horizon where the predictor is actually reliable (about 25 steps for LeWM), its accuracy advantage over exact simulation is small enough to disappear in noise.

The bottleneck is instead the planning method and its interaction with the horizon. Planning all 75 steps of a Push-T episode at once succeeds 22% of the time. Breaking that into three 25-step subgoal problems, with subgoals supplied by an expert, raises this to 80%. The model's planning horizon is the binding constraint, and the predictor's quality within that horizon barely moves the needle.

## A memory graph as a subgoal oracle

What happens when you don't have an expert providing subgoals? The paper tests a graph-search approach: embed demonstration states, build a graph over them, and find paths through the graph as subgoal sequences. Without any cost on edges, this succeeds 32% of the time — worse than expert subgoals — and produces physically impossible paths that jump between demonstrations. Adding a cost per edge, proportional to the distance between adjacent states, raised success to 85%, matching or slightly exceeding the expert-subgoal result.

The cost is doing a lot of work here. It enforces that paths through demonstration space stay physically plausible, and it rewards using long, continuous segments of real trajectories rather than stitching together distant fragments. The improvement from 32% to 85% by adding one term is a good illustration of how much implicit structure a planning objective needs to capture.

A further refinement — adding augmented memories by shifting, rotating, and mirroring demonstration segments — raised success on cases where real demonstrations give poor coverage. With nearby reference states removed (forcing the planner to rely on augmented data), this reached 85% versus 32% without augmentation.

## Goal specification as a hidden failure mode

The subtlest result is about goal images. Push-T goals are images of the final scene, which include the pusher's position. When the cost function scores the pusher position, the planner finds trajectories that satisfy the goal image literally — the pusher ends up in the right place, but the block doesn't. Excluding the pusher from the cost function flipped results from 9/50 to 39/50 on one test set.

This is a failure mode that compounds at the system level. The model predicts correctly. The planner searches correctly. The goal specification is wrong in a way that only becomes visible when you check whether the task was actually completed. On a harder test set (goals that no single demonstration directly connects), searching over possible pusher positions in the goal doubled success from 11% to 23%.

The implication is that world model planning pipelines need goals specified as sets of acceptable end states rather than single images — or at least goals where irrelevant variables are marginalized out. Neither of these is automatic, and both require knowing what's relevant in the state space.

## What the benchmark hides

The standard Push-T benchmark test set uses goals that lie along paths the existing demonstrations already travel. On this set, results look reasonable and don't vary much between methods. On a harder set where goals require combining segments from multiple demonstrations — requiring genuine generalization — results dropped from 75% to 45%, and sensitivity to random seeds increased substantially (38 out of 250 episodes flipped on seed change alone).

The harder test reveals that the best planner here reaches 56% success, and a learned hierarchical variant (H-JEPA) only matched or slightly improved on flat LeWM. Generalization beyond the demonstrations is still the open problem.

None of this is a reason to dismiss JEPA world models. It's a map of where the actual difficulty lives: not in the predictor, not in the learned representation, but in the planning infrastructure layered on top of it, and in the choice of goal representation. Those are solvable engineering problems — which is both encouraging and a reminder that benchmark numbers on standard test sets are a poor guide to them.
