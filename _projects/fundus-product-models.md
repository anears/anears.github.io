---
layout: page
title: Fundus screening product models
description: Deep learning models shipped in VUNO Med-Fundus AI (v1.2) and developed for its successor, VUNO Med-Fundus Pro
importance: 1
category: product
---

I was the individual owner of several deep learning models running inside **VUNO Med-Fundus AI**, a commercial 12-finding retinal fundus screening product, and its upcoming successor **VUNO Med-Fundus Pro**, a 3-disease (DR / AMD / RVO) screening product.

## Finding-classification model upgrade (Fundus AI v1.2)

Re-trained and re-designed the finding-classification models for the v1.2 product release:

- **AUROC +1.48%, AUPRC +2.22%** over the shipped models.
- Training pipeline overhauled — wall-clock training time reduced from **15 days to 8 hours**.

## Models for VUNO Med-Fundus Pro

- **Disease classification** — DR / AMD / RVO classifiers, including participation in the confirmatory clinical trial for regulatory approval.
- **Lesion quantification** — extended the single-class quantification model to a 4-class model.
- **Vessel + localization integration** — merged two separate models (vessel segmentation, optic disc/fovea localization) into one while maintaining accuracy, reducing inference cost.

All models were developed, validated, and handed over in release-ready form; see the [serving & release engineering](/projects/serving-release-engineering/) page for that side of the work.
