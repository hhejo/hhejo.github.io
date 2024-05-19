---
title: FP, FMP, FCP, LCP
date: 2024-05-06 09:26:47 +0900
last_modified_at: 2024-05-13 11:54:05 +0900
categories: [Web]
tags: [web, fp, fmp, fcp, lcp]
---

FP, FMP, FCP, LCP에 대해 각각 알아보겠습니다.

## 최초 페인트

최초 페인트 (First Paint, FP)

- 브라우저가 페이지 렌더링을 시작함
- 내비게이션과 브라우저가 처음으로 픽셀을 화면에 렌더링해서 본문에 기본 배경색과 시각적으로 다른 모든 항목 렌더링하는 사이의 시간

## 최초 의미있는 페인트

최초 의미있는 페인트 (First Meaningful Paint, FMP)

- FMP 이후에 웹 페이지 초기 렌더링 직후 기본 화면 레이아웃(above-the-fold)이 변경되고, 웹 글꼴이 로드됨
- FMP 이후 사용자는 웹 페이지를 유용하다고 느낌
- 페이지 로드의 작은 차이에도 매우 민감
- 이로 인해 일관되지 않은(이중적인) 결과가 발생할 수 있음
- 표준화할 수 없고 모든 웹 브라우저에서 구현되지 않았음

## 최초 콘텐츠풀 페인트

최초 콘텐츠풀 페인트 (First Contentful Paint, FCP)

- 콘텐츠가 포함된 첫 페인트
- 기본 콘텐츠가 완료된 시점을 측정
- 첫 번째 콘텐츠 요소가 렌더링될 때 모든 콘텐츠가 렌더링되는 것은 아님

FCP에 포함되는 시간

- FCP는 사용자가 처음 페이지로 이동한 시점부터, 페이지 콘텐츠의 일부가 화면에 렌더링된 시점까지의 시간을 측정
- 이전 페이지의 언로드 시간
- 연결 설정 시간
- 리디렉션 시간
- 첫 바이트까지의 시간 (TTFB)

이 측정 항목에서의 '콘텐츠'

- 텍스트
- 이미지(배경 이미지 포함)
- `<svg>` 요소
- 흰색이 아닌 `<canvas>` 요소

좋은 FCP 점수

- `1.8s` 이하이면 우수
- `3.0s` 보다 크면 좋지 않음

## 최대 콘텐츠풀 페인트

최대 콘텐츠풀 페인트 (Largest Contentful Paint, LCP)

- 콘텐츠가 포함된 최대 페인트
- 인지된 로드 속도를 측정하는 안정적인 Core Web Vitals 측정항목
- 페이지 로드 타임라인에서 페이지의 주요 콘텐츠가 로드되었을 가능성이 있는 지점을 표시
- LCP가 빠르면 사용자에게 페이지가 유용하다는 확신을 줄 수 있음

LCP에 포함되는 시간

- 사용자가 처음 페이지로 이동한 시점을 기준으로 표시 영역에 표시되는 가장 큰 이미지 또는 텍스트 블록의 렌더링 시간
- 이전 페이지의 언로드 시간
- 연결 설정 시간
- 리디렉션 시간
- 첫 바이트까지의 시간 (TTFB)

좋은 LCP 점수

- `2.5s` 이하이면 우수
- `4.0s` 보다 크면 좋지 않음

LCP를 결정할 때 아래 요소가 포함되어 측정됨

- `<img>` 요소
  - 첫 번째 프레임 프레젠테이션 시간은 GIF, 애니메이션 PNG와 같은 애니메이션 콘텐츠에 사용됨
- `svg` 요소 내의 `<image>` 요소
- `<video>` 요소
  - 포스터 이미지 로드 시간 또는 동영상의 첫 프레임 프레젠테이션 시간 사용 중 더 빠른 시간 적용
- `background-image`가 있는 요소
- `url()` 함수를 사용하여 로드된 배경 이미지가 있는 요소
  - CSS 그라데이션과 반대
- `<p>`와 같은 텍스트 노드 그룹
  - 텍스트 노드 또는 다른 인라인 수준 텍스트 요소 하위 요소를 포함하는 블록 수준 요소

LCP 측정에서는 일부 요소만 고려할 뿐 아니라 휴리스틱을 사용하여 사용자가 '콘텐츠가 없는' 것으로 간주할 가능성이 있는 특정 요소를 제외

- Chromium 기반 브라우저의 경우 다음이 포함됨
- 불투명도가 0인 요소로, 사용자에게 표시되지 않음
- 전체 표시 영역을 덮는 요소로, 배경 요소일 가능성이 높음
- 페이지의 실제 콘텐츠를 반영하지 않으며 엔트로피가 낮은 자리표시자 이미지 또는 기타 이미지

## 참고

[최초 페인트](https://developer.mozilla.org/ko/docs/Glossary/First_paint)

[최초 의미있는 페인트 (First Meaningful Paint, FMP)](https://developer.mozilla.org/ko/docs/Glossary/First_meaningful_paint)

[최초 콘텐츠풀 페인트 (First contentful paint, FCP)](https://developer.mozilla.org/ko/docs/Glossary/First_contentful_paint)

[LCP](https://developer.mozilla.org/ko/docs/Glossary/Largest_contentful_paint)

[First Contentful Paint (FCP)](https://web.dev/articles/fcp?hl=ko)

[Largest Contentful Paint (LCP)](https://web.dev/articles/lcp?hl=ko)

[콘텐츠가 포함된 첫 페인트](https://developer.chrome.com/docs/lighthouse/performance/first-contentful-paint?hl=ko)

[FMP(First Meaningful Paint)와 LCP(Largest Contentful Paint)](https://velog.io/@sjpark960520/FMPFirst-Meaningful-Paint와-LCPLargest-Contentful-Paint)
