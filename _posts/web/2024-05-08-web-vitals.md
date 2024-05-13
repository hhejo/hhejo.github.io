---
title: 웹 바이탈과 코어 웹 바이탈
date: 2024-05-08 09:25:56 +0900
last_modified_at: 2024-05-13 12:36:43 +0900
categories: [Web]
tags: [web, web-vitals, core-web-vitals, lcp, inp, cls]
---

웹 바이탈이란 무엇일까요? 코어 웹 바이탈과 그 구성 요소도 알아보겠습니다.

## 웹 바이탈

웹 바이탈 (Web Vitals)

- 웹에서 우수한 사용자 환경을 제공하는 데 필수적인 웹페이지 품질 신호에 관한 통합 가이드를 제공하기 위한 Google 이니셔티브
- 사용 가능한 다양한 성능 측정 도구를 단순화하고 사이트 소유자가 가장 중요한 측정항목인 코어 웹 바이탈에 집중할 수 있도록 돕는 것을 목표로 하는 지표

## 코어 웹 바이탈

코어 웹 바이탈 (Core Web Vitals)

- 모든 웹페이지에 적용되는 웹 바이탈의 하위 집합
- 모든 사이트 소유자가 측정해야 하고 모든 Google 도구에 표시됨
- 각 코어 웹 바이탈은 사용자 환경의 고유한 측면을 나타내며, 현장에서 측정 가능하며, 중요한 사용자 중심 결과의 실제 환경을 반영

### 코어 웹 바이탈을 구성하는 측정항목

1. Loading: LCP (Largest Contentful Pain)
2. Interactivity: INP (Interaction to Next Paint)
3. Visual Stability: CLS (Cumulative Layout Shift)

LCP (최대 콘텐츠 렌더링 시간)

- 로드 성능을 측정
- 우수한 사용자 환경을 제공하려면 페이지가 처음 로드되기 시작한 후 `2.5s` 이내에 LCP가 발생해야 함

INP (다음 페인트에 대한 상호작용)

- 상호작용을 측정
- 우수한 사용자 환경을 제공하려면 페이지의 INP가 `200ms` 이하여야 함
- FID를 대체함

CLS (누적 레이아웃 변경)

- 시각적 안정성을 측정
- 우수한 사용자 환경을 제공하려면 CLS를 `0.1` 이하로 유지해야 함

## 참고

[Web Vitals](https://web.dev/articles/vitals?hl=ko#lifecycle)

[코어 웹 바이탈 (Core Web Vitals)이란?](https://seo.tbwakorea.com/blog/core-web-vitals/)

[First Input Delay (FID)](https://web.dev/articles/fid?hl=ko)
