---
title: Tempo Overview
description: Grafana Tempo의 역할과 분산 추적 저장 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tempo, tracing
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# Tempo Overview

## 역할

Tempo는 분산 추적 데이터를 저장하고 검색하는 Trace Backend.

```text
Application
   ↓
OpenTelemetry
   ↓
OTel Collector
   ↓
Tempo
   ↓
Grafana
```

## Tempo가 하는 일

- Trace 저장
- Trace ID 기준 조회
- 서비스별 요청 흐름 확인
- Span별 처리 시간 확인
- Grafana 연동

## Tempo가 하지 않는 일

Tempo가 애플리케이션 요청을 직접 계측해서 Trace를 생성하는 것은 아님.

```text
Trace 생성/전파 → OpenTelemetry
Trace 수집/전달 → OTel Collector
Trace 저장/검색 → Tempo
Trace 시각화 → Grafana
```

## 확인 가능한 정보

```text
Trace ID: abc123

POST /recommend        4.9s
├─ Backend             100ms
├─ AI Request          4.7s
└─ Database Query      4.5s
```

요청 전체 시간과 각 Span의 처리 시간을 비교해 병목 구간 확인.

## 로그와 연결

Loki 로그와 Tempo Trace에 같은 `trace_id`가 있으면 Grafana에서 서로 이동 가능.

```text
Loki Log
trace_id=abc123
      ↕
Tempo Trace
Trace ID=abc123
```

## 참고

- Grafana Tempo: https://grafana.com/docs/tempo/latest/

다음: [Tempo Architecture](/Monitoring/Tempo/02-Architecture)
