---
layout: page
title: ARD-VLA
description: Asymmetric Residual Dual action heads for bimanual fine manipulation. 🚧 Under active development.
importance: 1
category: research
permalink: /projects/ard-vla/
---

**Status:** 🚧 under active development · paper in preparation · simulation stage

### Motivation

Existing bimanual VLA work (ACT/ALOHA, RDT-1B, OpenVLA-OFT, TwinVLA, SmolVLA) treats both arms symmetrically.
Humans use a natural role split: the non-dominant hand **stabilizes** an object while the dominant hand **acts** on it.
ARD-VLA builds this asymmetry into the model.

### Method

- **Base model:** SmolVLA (450M), fine-tuned with LoRA.
- **Dual action heads:** a _Stabilizer_ head and an _Actuator_ head share the same action-expert features but have separate parameters. Each is a 2-layer MLP, and together they add only about 0.12% of the model's parameters.
- **Asymmetric loss:** the stabilizer is trained for position accuracy and smoothness. The actuator also gets force and trajectory terms. The weights between these terms are balanced automatically with GradNorm.
- **Target hardware:** bimanual OpenArm with 16 DoF in total (7 joints and a 1-DoF gripper per arm).

### Efficiency work (toward on-device VLA)

- Profiled peak training memory with LoRA, bf16, and gradient checkpointing. A bimanual batch of 8 fits in about 2 GB.
- Measured 4-bit and 8-bit quantization on a real GPU. They save memory at small batch sizes, but the savings shrink or even reverse at large batches because activations dominate memory.
- Ran a layer-importance analysis of SmolVLA's default layer pruning.
- Implemented training-free visual token pruning that balances task relevance and diversity.

### Next

- Train and evaluate on real bimanual data, including the RoCo Challenge assembly tasks.
- Validate ARD-VLA 2.0's closed-loop judging and role-aware LAC with real weights ([details](/blog/2026/ard-vla-2/)).
