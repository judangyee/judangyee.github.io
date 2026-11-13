---
layout: page
title: ARD-Gen
description: Teleoperation-free data generation for bimanual fine manipulation
importance: 2
category: research
permalink: /projects/ard-gen/
github: https://github.com/judangyee/ARD-Gen
---

**Code:** [github.com/judangyee/ARD-Gen](https://github.com/judangyee/ARD-Gen)

Collecting bimanual demonstrations usually needs a teleoperation rig. ARD-Gen generates them in simulation instead.

### Pipeline

1. **Seed acquisition:** CMA-ES tunes admittance-controller gains until the task succeeds (MuJoCo, bimanual OpenArm).
2. **Role-specific augmentation:** the stabilizer arm gets geometric, MimicGen-style augmentation. The actuator arm gets diffusion-based augmentation.
3. **Language labeling:** episodes are labeled automatically from physics signals such as force, rotation, and direction.

### Results so far

- Peg-in-hole: 249 episodes (about 38.8k frames).
- Cap twisting: 209 episodes.
- Output is in a LeRobot-compatible format.
