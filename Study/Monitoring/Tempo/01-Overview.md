---
title: 01. Tempo Overview
description: Grafana Tempo 역할과 Trace 저장 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tempo, tracing
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 01. Tempo Overview

## 역할

Tempo = 분산 추적 데이터 저장 + 검색 Backend.

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

## 하는 일

- Trace 저장
- Trace ID로 검색
- 서비스별 요청 흐름 확인
- Span별 처리 시간 확인
- Grafana 연결

## 역할 구분

```text
Trace 생성/전파 → OpenTelemetry
Trace 수집/전달 → OTel Collector
Trace 저장/검색 → Tempo
Trace 확인      → Grafana
```

## 예시

```text
Trace ID: abc123

POST /recommend        4.9s
├─ Backend             100ms
├─ AI Request          4.7s
└─ Database Query      4.5s
```

전체 요청 시간과 Span 시간을 보고 병목 구간 확인.

## Loki와 연결

같은 `trace_id` 사용.

```text
Loki Log
trace_id=abc123
      ↕
Tempo Trace
trace_id=abc123
```

## 참고

- https://grafana.com/docs/tempo/latest/

## 다음

- [02. Tempo Architecture](./02-Architecture)
