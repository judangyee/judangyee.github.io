---
layout: post
title: When 4-bit quantization uses more memory than bf16
date: 2026-11-13
description: Measuring QLoRA-style quantization on SmolVLA for bimanual training
tags: on-device-vla quantization smolvla
categories: research-notes
---

I expected quantizing SmolVLA's backbone to save memory across the board. It didn't.

**Setup:** SmolVLM2-500M backbone with LoRA, ARD dual action heads, and gradient checkpointing. The input was a bimanual action chunk of 50 steps. I measured peak forward+backward GPU memory on Colab.

**What happened**

- At small batch sizes (1–8), 4-bit and 8-bit quantization saved roughly 3–26% compared with bf16.
- At large batch sizes (16–32), the savings disappeared. In some settings, 8-bit used **23% more** memory than bf16.

**Why**

- Quantization only shrinks the stored weights of the frozen backbone. As the batch grows, activations take up most of the memory instead.
- bitsandbytes also dequantizes weights on every forward pass, which adds overhead. This is especially costly for 8-bit LLM.int8().

**Takeaway:** quantization pays off for small-batch, low-VRAM settings, which is exactly the on-device case. On a GPU that can already fit large batches, it brings little benefit or even costs memory.
