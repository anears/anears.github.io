---
layout: page
title: ko-chart-vlm
description: 한국어 차트를 못 읽는 VLM을 7일 만에 진단 → 파인튜닝 → 서빙까지 풀사이클로 고친 사이드 프로젝트 (val 70.2% → 96.6%)
img: assets/img/ko-chart-vlm.png
importance: 1
---

오픈소스 VLM(**Qwen3-VL-8B**)이 한국어 차트를 얼마나 읽는지 **숫자로 측정**하고, 약점을 겨냥한 합성 데이터로 **QLoRA 파인튜닝**해 **val 정확도 70.2% → 96.6%(+26.4%p)** 로 끌어올린 뒤, **vLLM 서빙**까지 진행한 7일 사이드 프로젝트입니다.

<div class="row justify-content-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ko-chart-vlm.png" title="한국어 단위 환산 약점 진단용 차트" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    진단에 쓴 차트 예시 — "연구개발 예산은 몇 억 원인가요?"(차트엔 0.85조)에 zero-shot 모델은 850억 원이라고 답했다(10배 오류). 1조 = 10,000억이라는 한국어 단위 체계가 비어 있었다.
</div>

## 무엇을 했나

- **Day 1 · 진단** — 수기 차트 8장·질문 16개로 모델이 무너지는 지점을 특정: ① 조→억 단위 환산(0%) ② 값 라벨 없는 조밀 차트의 순위 판단 ③ 근소한 값 비교.
- **Day 2 · 합성 데이터** — matplotlib + 나눔고딕으로 약점을 겨냥한 차트 2,000장, **5,671 QA쌍** 생성. 정답은 원본 데이터에서 계산(하드코딩 없음), train/val 시드 분리로 누수 차단.
- **Day 3 · 평가 벤치마크** — 한국어 단위를 정규 원-환산값으로 바꿔 "8,500억"·"0.85조"·"850000000000"을 같은 잣대로 채점하는 strict/relaxed 채점기 설계(자체검증 20/20).
- **Day 4 · QLoRA 파인튜닝** — LLaMA-Factory, 4bit, LLM 디코더만 학습(전체 파라미터의 **0.5%**). RTX 4090 한 장으로 약 1시간 40분.
- **Day 5 · 오류 분석·ablation** — 조→억 환산 0% → 96%, 조밀 차트 순위 57% → 95%. **1 epoch가 3 epoch의 ~99%를 1/3 시간에 달성**하는 것도 확인.
- **Day 6 · 서빙** — 어댑터 병합 후 **vLLM** 원본·파인튜닝 나란히 비교하는 Gradio 데모. **AWQ W4A16** 양자화로 17GB → 6.8GB.

## 정직하게 기록한 한계

- **합성 SFT는 렌더링 분포에 과적합** — 분포 밖 수기 차트에선 75% → 81%로 부분만 전이됐고, 환산 오류는 방향만 뒤집혔습니다(10배 과소 → 10배 과대).
- **오프라인 정확도가 서빙으로 그대로 이어지지 않음** — 같은 가중치인데 transformers 평가와 vLLM 서빙의 이미지 전처리 경로(resize factor 28 vs 32)가 달라 답이 달라지는 **train/serve preprocessing skew**를 서빙 단계에서 실제로 잡았습니다.

## 링크

- GitHub: [anears/ko-chart-vlm](https://github.com/anears/ko-chart-vlm) — 일자별 리포트·코드·wandb 로그
- 정리 글: [한국어 차트를 못 읽는 VLM, 7일 만에 고치기](https://github.com/anears/ko-chart-vlm/blob/main/docs/blog.md)

**스택**: Qwen3-VL-8B · LLaMA-Factory (QLoRA) · vLLM · llmcompressor (AWQ) · uv
