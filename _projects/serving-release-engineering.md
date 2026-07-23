---
layout: page
title: Model serving & release engineering
description: Docker-packaged inference runtimes, asynchronous serving workers, and CI/CD release pipelines for the fundus products
importance: 3
category: product
---

Alongside model development, I owned the engineering that turns trained models into release-ready runtimes:

- **Inference runtime** — designed and implemented asynchronous inference workers (**Celery + Redis**), packaged as **Docker** images and handed over to the product development team, which operates the service servers.
- **CI/CD** — built and maintained the release pipelines (GitHub Actions), including **model-weight encryption** in the packaging step and smoke tests on release candidates.
- **Inference optimization** — applied **ONNX / OpenVINO** conversion and **INT8 quantization** to the shipped models.
- **Benchmarking** — built a benchmark harness used to validate release candidates on a pool of ~13,700 images.
