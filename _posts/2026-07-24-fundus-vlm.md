---
layout: post
title: fundus-vlm — 오픈 VLM으로 안저 스크리닝 리포트 만들기
date: 2026-07-24 09:00:00+0900
description: 오픈 VLM의 안저 판독 능력을 정량 측정하고 QLoRA로 전문 CNN급까지 끌어올린 뒤 구조화 리포트 생성·서빙까지 — 공개 데이터만으로
tags: vlm qlora medical-ai fundus side-project
related_posts: false
---

[지난 프로젝트(ko-chart-vlm)](/blog/2026/ko-chart-vlm/)에서 "오픈 VLM의 약점을 숫자로 측정 → 겨냥한 데이터로 파인튜닝 → 서빙"이라는 사이클을 한국어 차트로 돌려봤습니다. **[fundus-vlm](https://github.com/anears/fundus-vlm)**은 같은 방법론을 제 연구 도메인인 **컬러 안저(color fundus) 사진**으로 이식한 후속 프로젝트입니다.

원칙은 하나 추가했습니다: **공개 데이터셋만 사용합니다** (APTOS 2019 · Messidor-2 · IDRiD · REFUGE · ODIR-5K · RFMiD, 총 ~31GB). 재현과 공유가 가능해야 하고, 내부/비공개 데이터는 쓰지 않습니다.

## 결과 하이라이트 (frozen val 319문항)

| 태스크 | 지표 | Qwen3-VL-8B | +QLoRA | MedGemma-4B | +QLoRA | 전문 CNN |
| --- | --- | --- | --- | --- | --- | --- |
| DR 등급 | QWK | 0.718 | 0.862 | 0.838 | **0.874** | 0.861 |
| CDR 추정 | Pearson r | 0.216 | −0.00 | 0.368 | **0.737** | — |
| 다질환 8종 | macro-F1 | 0.236 | 0.487 | 0.194 | **0.558** | — |

- 파인튜닝한 VLM이 **DR 전용 CNN(EfficientNet, QWK 0.861)의 상한선을 대등/상회**하면서, 다질환 스크리닝·CDR 정량 추정·구조화 리포트 생성까지 **한 모델**로 처리합니다.
- 최종 서빙 모델은 **MedGemma-4B + QLoRA** — 4B 의료 특화 모델이 8B 일반 모델을 전 태스크에서 이겼습니다.

## 가장 흥미로웠던 발견 — 의료 사전학습이 정량 회귀의 분기점

CDR(수직 컵-디스크 비율, 녹내장 지표) 추정에서 두 모델의 운명이 갈렸습니다:

- **Qwen3-VL-8B(일반)**: zero-shot에서 실제 값과 무관하게 0.5 근처 상수를 출력했고(r=−0.14), SFT 후에는 **평균값으로 붕괴**(r→0)했습니다. 분류는 배우지만 정량 읽기는 배우지 못한 겁니다.
- **MedGemma-4B(의료)**: 같은 340개 CDR 학습 샘플로 **r 0.368 → 0.737**까지 실제로 학습했습니다.

같은 데이터, 같은 레시피인데 **의료 시각 사전학습 유무가 소량 데이터 정량 회귀의 성패를 갈랐습니다.** disc 영역을 crop해서 주는 ablation은 도움이 되지 않았어서(음성 결과), 결정 요인은 입력 해상도가 아니라 사전학습이라는 쪽에 무게가 실립니다.

## 진행 방식 — 6개 phase

1. **데이터** — 매니페스트 기반 다운로더로 공개 6종 다운로드·검증. 미러 데이터의 함정(APTOS 라벨이 클래스명 알파벳순으로 인코딩되어 임상 순서와 다름)을 검증노트로 기록.
2. **zero-shot 진단** — 170문항으로 약점 정량화: Mild DR **0/18**, glaucoma **0/5**, CDR 상수 출력.
3. **평가 하네스** — frozen val 319문항 + 채점기 자체검증 17/17. 일반(Qwen) vs 의료(MedGemma) baseline 확정.
4. **데이터 구축** — train 여집합에서 VQA + 템플릿 리포트 **6,528페어** 생성 (val 누수 없음, 약점 타깃 균형 샘플링, 전문가 라벨·세그멘테이션에서 GT 계산).
5. **QLoRA SFT** — 두 모델 병렬 파인튜닝 (RTX 4090 한 장).
6. **분석·서빙** — 오류분석, EfficientNet 상한선 비교, disc-crop ablation, Gradio 데모 + vLLM/AWQ 서빙 문서.

ko-chart-vlm에서 가져온 실험 규칙도 그대로입니다 — 실험 1건 = 디렉터리 1개(결과 jsonl + meta + report), 모든 주장은 수치로, 실패(Qwen CDR 붕괴, disc-crop 음성)도 리포트에 그대로 남깁니다.

## 링크

- 저장소: [anears/fundus-vlm](https://github.com/anears/fundus-vlm) — phase별 리포트·데이터 검증노트·데모 코드
- 선행 프로젝트: [한국어 차트를 못 읽는 VLM, 7일 만에 고치기](/blog/2026/ko-chart-vlm/)

**스택**: Qwen3-VL-8B · MedGemma-4B · QLoRA(커스텀 트레이너) · vLLM · Gradio · uv
