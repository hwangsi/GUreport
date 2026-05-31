---
id: radiology-gureport
title: 전립선 MRI 구조화 판독문 작성 보조
specialty: radiology
patterns: [report-drafting, korean-clinical-text]
author: Sungil Hwang
tool: [claude-api]
model: claude-opus-4-8
status: draft
data: synthetic
---

# 전립선 MRI 구조화 판독문 작성 보조

> 한 줄 요약: 전립선 MRI 소견을 PI-RADS 구조에 맞춘 구조화 판독문 초안으로 정리한다.

## 문제 — 왜 만들었나

전립선 MRI 판독문을 PI-RADS 구조(zone별 평가, 병변 위치, 최종 카테고리)에 맞춰
일관되게 작성하는 데 반복 노력이 든다. 판독의마다 서술 순서·표현이 달라
보고 품질과 후속 분석 일관성이 떨어진다.

## 무엇을 하나

자유서술 소견 입력 → PI-RADS 구조에 맞춘 구조화 판독문 초안 출력.
최종 진단·점수의 임상 판단은 사람이 한다(초안 작성만 보조).

## 레시피

    [GUreport에서 실제로 쓰는 검증된 프롬프트를 여기에 붙여넣으세요]

## 검증

(아직 정리 전 — 적용 건수·일치율 등 확인되면 채움)

## 주의사항 · 안전

- 실제 환자정보는 넣지 않음. 비식별/합성 데이터만 사용.
- 임상 판단을 대체하지 않음. 판독의 최종 검토 필수.
