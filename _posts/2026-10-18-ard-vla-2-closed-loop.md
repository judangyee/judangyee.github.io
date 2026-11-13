---
layout: post
title: "ARD-VLA 2.0: closing the loop with a shared-backbone judge"
date: 2026-10-18
description: Adding success judgment, step selection, and recovery to a bimanual VLA without a second model
tags: ard-vla closed-loop task-planning smolvla
categories: research-notes
---

## The problem: the original model is open-loop

The first version of [ARD-VLA](/projects/ard-vla/) only produces actions. It never checks whether a step succeeded.
If a grasp slips or an insertion misses, the model doesn't notice. It moves on to the next step from a failed state and has no way to retry.

So I upgraded it to **judge success at every step and plan around failures**.

## Architecture: one backbone, three branches

The judge is inspired by **Laya**, a "System 1" model that outputs only success/failure probabilities instead of generating text.
Running a second VLM backbone just for judging would be expensive, so I kept SmolVLA's **SmolVLM2** backbone and borrowed only Laya's output format.

A single backbone pass reads the inputs once. Three branches share its features:

| Branch                        | Role        | Output                                                                                                |
| ----------------------------- | ----------- | ----------------------------------------------------------------------------------------------------- |
| **Judge head**                | what to do  | calibrated success probability, one of 5 failure types, progress (0–1); no gradient into the backbone |
| **Selector head**             | what to do  | picks one allowed candidate: next step, a recovery, or stop                                           |
| **Action expert + ARD heads** | how to move | 50-step action chunk via flow matching, split into Stabilizer (left arm) and Actuator (right arm)     |

The inputs are the camera image, an instruction (goal, current subtask, and completed steps), and the robot state (16 DoF: 7 joints + 1 gripper per arm).

On top sits a **decision layer**. It is a set of rules, not a trained module, and it outputs one of `ADVANCE`, `RETRY`, `RECOVER`, `RECHECK`, or `STOP`.
That decision sets the next cycle's instruction and whether the chunk gets executed. A safety rule restricts it to allowed candidates and stops the robot once a retry limit is exceeded.

## One cycle

1. Observe and build the instruction for the current node.
2. Run **one backbone forward pass**, which produces the action chunk and the Judge/Selector outputs.
3. If the previous subtask just ended, the decision layer acts:
   - `RETRY`: execute this chunk as is.
   - `ADVANCE` / `RECOVER`: the instruction changes, so observe again.
   - `RECHECK`: observe again without executing.
   - `STOP`: move to a safe pose and end.
4. Otherwise, execute the chunk step by step until the subtask ends or its step budget runs out.

Judgment happens **only after a subtask ends**, based on what the robot observes after executing it.
While a subtask is in progress, the cost is one forward pass per chunk. `ADVANCE` and `RECOVER` add one extra pass because the instruction changes. So "one backbone pass" holds per chunk; actual end-to-end latency still has to be measured with real weights.

## Task planning in three layers

| Layer                      | When               | Who                                     | What it does                                                                                   |
| -------------------------- | ------------------ | --------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Task graph**             | before execution   | a human (for fixed processes) or an LLM | defines subtasks, their order, entry/exit conditions, allowed recoveries, and retry limits     |
| **In-execution selection** | after each subtask | decision layer + Selector               | chooses the next node, a recovery, a recheck, or stop, but only among options the graph allows |
| **Replanning**             | after `STOP`       | a human or an LLM                       | edits the graph itself from a safe pose (add steps, reorder, change recovery methods)          |

For example, a peg-in-hole graph is `align → insert → done`. Each step has recovery candidates:

- R1: redo the same step
- R2: return to the start pose
- R3: retry with different settings

At first a rule table picks the recovery; later the trained Selector picks it from the scene.
If the limit is exceeded, the robot executes **R4 STOP**: it holds a safe pose, and the graph is fixed one level up.

## How this differs from existing approaches

| Compared with                                 | Their limitation                                                              | ARD-VLA 2.0                                                                                    |
| --------------------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Open-loop VLAs (SmolVLA, OpenVLA, ARD-VLA v1) | never check success; move on from failed states                               | judges every step and retries the same step with a different method                            |
| LLM planner + VLA                             | replanning mid-execution takes seconds; can hallucinate steps or break format | judging and selection fit inside one backbone pass; nothing is generated, so no invented steps |
| Separate VLM success detector                 | a second model adds memory and latency                                        | shares the policy's backbone and adds only small heads, so it targets a single consumer GPU    |
| Symmetric bimanual VLAs                       | can't tell which arm caused a failure                                         | judges right-arm failures and left-arm slips separately, carrying role asymmetry into judging  |

**Caveat:** these are design-level differences. The structure is implemented and passes tests with a tiny synthetic model, but it hasn't been trained or measured with real weights yet. Running it in Isaac Sim is the next step once I have the hardware.

Code: [judangyee/ARD-VLA (branch `closed-loop-structure`)](https://github.com/judangyee/ARD-VLA)
