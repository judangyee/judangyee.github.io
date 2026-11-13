---
layout: about
title: about
permalink: /
subtitle: CS Undergraduate · Kyonggi University · Independent VLA Researcher

profile:
  align: right
  image: # add a photo at assets/img/prof_pic.jpg and set it here
  image_circular: false
  more_info: >
    <p>Seoul, South Korea</p>

selected_papers: false
social: true

announcements:
  enabled: true
  scrollable: true
  limit: 5

latest_posts:
  enabled: true
  scrollable: true
  limit: 3
---

I'm a computer science undergraduate doing independent research on **Vision-Language-Action (VLA) models** for robotic manipulation.

Most bimanual robot learning treats both arms symmetrically. Humans don't: one hand stabilizes while the other acts.
My main project, **[ARD-VLA](/projects/ard-vla/)**, builds that role asymmetry into a VLA, with separate _Stabilizer_ and _Actuator_ action heads on top of SmolVLA.
It targets fine, force-sensitive tasks such as peg-in-hole insertion and cap twisting.

To get training data without a teleoperation rig, I built **[ARD-Gen](/projects/ard-gen/)**, a data generation pipeline for bimanual fine manipulation on a simulated OpenArm.
It finds seed trajectories with CMA-ES, augments them differently for each arm's role, and labels them with language automatically.

I'm also interested in **on-device VLA**: making these models small and fast enough to run on the robot itself.
I've worked with LoRA, 4/8-bit quantization, gradient checkpointing, layer-importance analysis, and visual token pruning.

Long term, I want to build VLA systems that handle dexterous tool use and bimanual work in manufacturing.

**Research interests:** Vision-Language-Action models · Bimanual manipulation · Tool use · On-device & efficient VLA · Robot learning
