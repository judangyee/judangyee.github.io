---
layout: post
title: "Role-aware LAC: anti-shake filtering for asymmetric arms"
date: 2026-10-25
description: Adapting Co-VLA's training-free action filter to ARD-VLA's stabilizer/actuator split
tags: ard-vla bimanual action-filtering co-vla
categories: research-notes
---

## Co-VLA's LAC

[Co-VLA](https://arxiv.org/abs/2606.20285) (June 2026) proposes several ideas for coordinated bimanual control.
The part I focused on is **LAC (Latent-Aware Controller)**, a training-free filter applied at inference time.

The easiest way to think about LAC is **image stabilization for robot arms**.

## How it works

LAC splits each bimanual command into two parts:

- a **shared motion** that both arms make together, and
- a **residual motion** that each arm makes on its own.

It then asks whether the residual is meaningful or just jitter.
Based on the answer, it decides at every step whether to follow the new command immediately or hold the previous motion and follow slowly.

It distinguishes three cases:

| What the arms do                | Example                                   | LAC's call               | Action        |
| ------------------------------- | ----------------------------------------- | ------------------------ | ------------- |
| move **together**               | carrying a desk                           | large shared motion      | execute as is |
| push in **opposite directions** | holding a bottle and twisting off the cap | intentional coordination | keep it       |
| shake **independently**         | hands trembling                           | jitter                   | smooth it out |

The middle case is the key. When the two hands apply opposing forces, as in opening a bottle, LAC recognizes deliberate coordination and doesn't filter it away.

## Going one step further with arm roles

ARD-VLA assigns each arm a role from the start, so the filter can treat them differently:

- The **Stabilizer** arm (holding the bottle) always suppresses jitter strongly and stays firm.
- The **Actuator** arm (twisting the cap) responds quickly and follows the command as is during coordinated motion.

It's like a person holding a nail steady with one hand while swinging a hammer with the other.

## Implementation

The implementation reuses ARD-VLA's existing structure.
ARD-VLA builds each command as a **base action plus a role-specific residual**. I took those two terms out directly and used them as LAC's shared motion and residual motion.

- LAC sits at the point where commands are sent to the arms one step at a time (`select_action()`).
- A single config option selects **off / original LAC / role-aware LAC**.
- Gripper open/close commands pass through without filtering.

**Status:** implemented on the `lac-inference` branch and not yet connected to the 2.0 closed-loop path. It hasn't been validated with real weights or in simulation.

Code and a detailed write-up: [judangyee/ARD-VLA](https://github.com/judangyee/ARD-VLA)
