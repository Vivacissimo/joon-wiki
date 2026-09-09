---
title: 02. Jaeger Architecture
description: OpenTelemetry에서 Jaeger까지 Trace 흐름
published: true
date: 2026-09-09T12:04:00.000Z
tags: monitoring, jaeger, tracing, opentelemetry
editor: markdown
dateCreated: 2026-09-09T12:04:00.000Z
---

# 02. Jaeger Architecture

## 전체 흐름

```text
Application
   ↓
OpenTelemetry SDK / Auto Instrumentation
   ↓
Trace / Span
   ↓
OTel Collector
   ↓
Jaeger
   ├─ Trace 저장
   ├─ Trace 검색
   └─ Jaeger UI
```

## Application

서비스에서 Trace / Span 생성.

```text
Backend
trace_id = abc123
span_id  = span-a

AI
trace_id = abc123
span_id  = span-b
```

서비스가 달라도 같은 `trace_id`를 이어가야 요청 하나로 묶임.

## OpenTelemetry

Trace를 만들고 다음 서비스로 Trace Context 전달.

```text
Backend
trace_id=abc123
    ↓
AI
trace_id=abc123
    ↓
DB
trace_id=abc123
```

## OTel Collector

Trace 데이터 중간 수집/전달.

```text
Application
   ↓ OTLP
OTel Collector
   ↓
Jaeger
```

Jaeger로 바로 보내는 구조도 가능하지만 Collector를 중간에 두면 전송 대상 변경이나 필터링을 한곳에서 처리하기 편함.

## Jaeger

받은 Trace를 저장하고 조회.

```text
Trace abc123
├─ Backend
├─ AI
└─ DB
```

Jaeger UI에서 Trace ID 검색, Span 시간, 에러 여부 확인.

## 로그와 연결

로그에도 같은 `trace_id`를 넣어두면 Trace와 로그를 맞춰볼 수 있음.

```json
{
  "level": "ERROR",
  "service_name": "ai",
  "trace_id": "abc123",
  "message": "pgvector query timeout"
}
```

```text
Jaeger
Trace abc123
    ↓
같은 trace_id 검색
    ↓
Loki
ERROR pgvector query timeout
```

## Tempo를 쓸 때와 차이

```text
OpenTelemetry
    ↓
OTel Collector
    ├─→ Tempo  → Grafana
    └─→ Jaeger → Jaeger UI
```

Trace Backend를 Tempo로 정했으면 Jaeger는 빼도 됨.

Jaeger를 쓰기로 했으면 Tempo 대신 Jaeger를 Trace 저장소로 두면 됨.

## 앞 문서

- [01. Jaeger Overview](./01-Overview)
