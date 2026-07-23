---
layout: page
title: Input-filtering (OOD) model redesign
description: Rebuilt the product's out-of-distribution input filter — false rejections of normal images down from 1,359 to 21
importance: 2
category: product
related_publications: true
---

Fundus screening products must reject inputs that are not gradable fundus photographs (other imaging modalities, corrupted captures) **without** rejecting valid ones. The original one-class formulation over-rejected valid fundus images in the field.

As the individual owner, I redesigned the input filter end-to-end:

- Reformulated the problem from **one-class anomaly detection to binary classification**.
- Collected and curated non-fundus negatives across modalities (ultra-widefield, MRI, X-ray, CT) for re-training.
- Result: false rejections of normal fundus images reduced from **1,359 to 21** on the validation pool, with specificity improving **0.9806 → 0.9997** (+1.91%p).
- Shipped in the v1.2 product release.

This was a product engineering effort, independent from my academic research on OOD detection ({% cite kwon2023shift %}).
