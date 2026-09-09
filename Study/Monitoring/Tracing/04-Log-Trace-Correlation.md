---
title: 04. Log Trace Correlation
description: Loki 로그와 Tempo Trace를 Trace ID로 연결하는 방식
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, loki, tempo, grafana
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 04. Log Trace Correlation

## 목적

로그와 Trace를 따로 보는 것이 아니라 같은 요청을 기준으로 서로 연결해서 장애 원인을 찾는 구조.

핵심 연결 값은 `trace_id`.

```text
Loki Log
trace_id=abc123
      ↕
Tempo Trace
Trace ID=abc123
```

## 예시

Grafana에서 5xx 로그 발견.

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

관련 AI 로그 확인.

```text
trace_id=abc123
WARN  pgvector slow query
ERROR pgvector query timeout
```

## 장애 분석 흐름

```text
Prometheus Metric
"P95 Latency 증가"
        ↓
Tempo Trace
"DB Query 구간 4.5초"
        ↓
Loki Log
"pgvector query timeout"
```

Metric은 `무슨 문제가 발생했는지`, Trace는 `어디에서 발생했는지`, Log는 `왜 발생했는지` 찾는 데 사용.

## 필요한 조건

### 로그

JSON 로그에 Trace Context 포함.

```text
trace_id
span_id
```

### Trace

OpenTelemetry를 통해 서비스 간 Context Propagation이 정상 동작해야 함.

### Grafana

- Loki Data Source 연결
- Tempo Data Source 연결
- Loki의 trace_id에서 Tempo Trace로 이동할 수 있도록 연결
- 필요하면 Tempo Span에서 관련 Loki 로그를 검색하도록 연결

## 최종 확인 화면

최소 검증 목표

1. Grafana에서 ERROR 로그 선택
2. 로그의 `trace_id` 확인
3. 동일 Trace를 Tempo에서 열기
4. Backend → AI → DB Span 흐름 확인
5. 특정 Span과 관련된 로그 조회

이 흐름이 동작하면 로그 수집과 분산 추적의 상관관계가 연결된 상태로 볼 수 있음.

## 참고

- Grafana Tempo Trace Visualization: https://grafana.com/docs/tempo/latest/visualize-traces/
