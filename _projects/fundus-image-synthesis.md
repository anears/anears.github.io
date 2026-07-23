---
layout: page
title: Generative models for fundus image synthesis
description: Text-conditioned synthesis at 1.2M-image scale and mask-conditioned synthesis for extreme low-label segmentation
importance: 4
category: research
---

Two independent lines of work on conditional generative models for retinal fundus images:

## Text-to-fundus

Built a large-scale text-conditioned fundus synthesis pipeline: constructed a large fundus dataset, converted labels into text descriptions, and trained a **SANA**-based text-to-image model, generating **~1.2M synthetic fundus images**.

- Key result: classifiers trained **only on synthetic images performed on par with those trained only on real images** — evidence that synthetic data can substitute for real data where sharing or collection is constrained.

## Mask-to-fundus

For a lesion segmentation task with only **54 labeled images**, built a mask-conditioned synthesis pipeline with a **two-stage recipe** — pretraining on large public data, then **LoRA** fine-tuning on the 54 target images — and used the synthesized image–mask pairs as augmentation.

- Improved 4-class segmentation **AUPRC by +1.75%p**.

A related patent (image style transfer, KR 10-2592666) is registered.
