---
title: 04. Log Trace Correlation
description: Loki 로그와 Tempo Trace를 Trace ID로 연결
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, loki, tempo, grafana
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 04. Log Trace Correlation

## 개념

Log와 Trace를 같은 요청 기준으로 연결해서 보는 것.

연결 키 = `trace_id`

```text
Loki Log
trace_id=abc123
      ↕
Tempo Trace
trace_id=abc123
```

## 예시

Grafana에서 ERROR 로그 확인.

```json
{
  "level": "ERROR",
  "service_name": "backend",
  "event": "recommendation_failed",
  "trace_id": "abc123",
  "message": "AI request timeout"
}
```

같은 Trace ID를 Tempo에서 확인.

```text
Trace abc123

POST /recommend        4.9s
├─ Backend             100ms
└─ AI Request          4.7s
   └─ DB Query         4.5s
```

같은 요청의 로그 확인.

```text
trace_id=abc123
WARN  pgvector slow query
ERROR pgvector query timeout
```

## 장애 볼 때 흐름

```text
Prometheus
P95 Latency 증가
        ↓
Tempo
DB Query 4.5초
        ↓
Loki
pgvector query timeout
```

```text
Metric → 무슨 문제가 생겼나
Trace  → 어디에서 생겼나
Log    → 왜 생겼나
```

## 필요한 것

### Log

```text
trace_id
span_id
```

JSON 로그에 포함.

### Trace

OpenTelemetry로 서비스 간 Context가 이어져 있어야 함.

### Grafana

- Loki Data Source
- Tempo Data Source
- Log의 `trace_id` → Tempo Trace 연결
- Tempo Span → 관련 Loki Log 연결

## 확인 순서

1. Grafana에서 ERROR 로그 선택
2. `trace_id` 확인
3. 같은 Trace를 Tempo에서 열기
4. Backend → AI → DB Span 확인
5. 해당 Span 관련 로그 확인

여기까지 연결되면 로그 수집 + 분산 추적 상관관계 확인 가능.

## 참고

- https://grafana.com/docs/tempo/latest/visualize-traces/

## 다음

- [Grafana](../Grafana/Install)
