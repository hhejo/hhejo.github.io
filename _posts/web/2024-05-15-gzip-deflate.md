---
title: gzip과 DEFLATE
date: 2024-05-15 19:43:09 +0900
last_modified_at: 2024-05-19 13:19:32 +0900
categories: [Web]
tags: [web, gzip, deflate]
---

gzip이란 무엇일까요? 또, DEFLATE와 어떤 관계가 있을까요

## gzip

gzip (GNU zip)

- 파일 압축에 쓰이는 응용 소프트웨어
- ZIP과 같이 DEFLATE 알고리즘을 따르나 여러 파일을 하나의 파일로 압축하는 옵션이 없음
- 여러 파일 또는 디렉터리를 하나의 파일로 압축하기 위해 gzip은 보통 Tar와 같이 사용됨

## DEFLATE

DEFLATE

- ZIP, gzip 등의 프로그램에서 사용되는 무손실 압축 데이터 포맷이자 알고리즘
- 압축률에 비해 압축/해제 속도가 빠름
- 나중에 나온 알고리즘에 비해서는 압축률이 다소 떨어짐

## 참고

[gzip](https://ko.wikipedia.org/wiki/Gzip)

[DEFLATE](https://ko.wikipedia.org/wiki/DEFLATE)
