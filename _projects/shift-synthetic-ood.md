---
layout: page
title: SHIFT — synthetic outlier exposure for OOD detection
description: Zero-shot synthetic outlier generation with visual foundation models (BMVC 2023, oral)
importance: 3
category: research
related_publications: true
---

Outlier-exposure methods for out-of-distribution detection need real outlier data — which is expensive to collect and curate. **SHIFT** generates synthetic outliers **zero-shot** by combining CLIP-based semantic masking with latent-diffusion inpainting, so that the synthesized images stay near the in-distribution manifold while removing class-defining semantics.

- Improved OOD detection **AUROC by +0.9 to +5.9%p** across CIFAR-10 / CIFAR-100 / STL-10 benchmarks.
- Selected for **oral presentation at BMVC 2023** (equal-contribution first author); code released {% cite kwon2023shift %}.
