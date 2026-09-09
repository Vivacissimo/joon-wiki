---
title: Monitoring
description: Metrics, Logs, Traces 기반 Observability 정리
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, observability
editor: markdown
dateCreated: 2026-06-04T07:48:45.722Z
---

# Monitoring & Observability

## 개요

Metrics, Logs, Traces를 같이 봐서 시스템 상태 확인 + 장애 원인 추적.

```text
                    Grafana
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Prometheus         Loki           Tempo
    Metrics           Logs           Traces
```

| 데이터 | 보는 내용 | 도구 |
|---|---|---|
| Metrics | 요청량, 에러율, 지연시간, 자원 상태 | Prometheus |
| Logs | 어떤 오류가 발생했는지, 왜 실패했는지 | Loki |
| Traces | 요청이 어디를 거쳤고 어디서 느려졌는지 | Tempo |
| 화면 | Metric / Log / Trace 조회 및 연결 | Grafana |

장애 볼 때 흐름

```text
Metric
"문제가 생겼나?"
   ↓
Trace
"어디에서 생겼나?"
   ↓
Log
"왜 생겼나?"
```

---

# 보는 순서

```text
Monitoring
   │
   ▼
1. Logging
   01 Overview
      ↓
   02 Log Data Dictionary
      ↓
   03 Structured JSON Log
      ↓
   04 Log Collection Pipeline
   │
   ▼
2. Loki
   01 Overview
      ↓
   02 Architecture
   │
   ▼
3. Distributed Tracing
   01 Overview
      ↓
   02 Trace and Span
      ↓
   03 Context Propagation
   │
   ▼
4. OpenTelemetry
   01 Overview
      ↓
   02 Collector
   │
   ▼
5. Tempo
   01 Overview
      ↓
   02 Architecture
   │
   ▼
6. Log ↔ Trace Correlation
   Tracing / 04
   │
   ▼
7. Grafana
```

정리하면

1. **Logging** — 어떤 로그를 남길지 + JSON 형식
2. **Loki** — 로그 저장/검색
3. **Distributed Tracing** — Trace, Span, Context 개념
4. **OpenTelemetry** — Trace 생성/전파 + Collector
5. **Tempo** — Trace 저장/검색
6. **Log Trace Correlation** — `trace_id`로 Loki ↔ Tempo 연결
7. **Grafana** — Metric / Log / Trace 한 화면에서 확인

`Tracing/04-Log-Trace-Correlation`은 폴더상 04번이지만 OpenTelemetry와 Tempo까지 보고 나서 보는 순서.

---

# 문서 구조

## Logging

1. [01. Logging Overview](./Monitoring/Logging/01-Overview)
2. [02. Log Data Dictionary](./Monitoring/Logging/02-Log-Data-Dictionary)
3. [03. Structured JSON Log](./Monitoring/Logging/03-JSON-Log-Format)
4. [04. Log Collection Pipeline](./Monitoring/Logging/04-Collection-Pipeline)

## Loki

1. [01. Loki Overview](./Monitoring/Loki/01-Overview)
2. [02. Loki Architecture](./Monitoring/Loki/02-Architecture)

## Distributed Tracing

1. [01. Distributed Tracing Overview](./Monitoring/Tracing/01-Overview)
2. [02. Trace and Span](./Monitoring/Tracing/02-Trace-Span)
3. [03. Context Propagation](./Monitoring/Tracing/03-Context-Propagation)
4. [04. Log Trace Correlation](./Monitoring/Tracing/04-Log-Trace-Correlation)

## OpenTelemetry

1. [01. OpenTelemetry Overview](./Monitoring/OpenTelemetry/01-Overview)
2. [02. OpenTelemetry Collector](./Monitoring/OpenTelemetry/02-Collector)

## Tempo

1. [01. Tempo Overview](./Monitoring/Tempo/01-Overview)
2. [02. Tempo Architecture](./Monitoring/Tempo/02-Architecture)

## Grafana

- [Grafana](./Monitoring/Grafana/Install)

## Kiali

- [Kiali](./Monitoring/kiali/what-is-kiali)

---

# 전체 데이터 흐름

```text
Application
   │
   ├─ Metrics ───────────────→ Prometheus ─┐
   │                                      │
   ├─ JSON Log → Collector → Loki ────────┤
   │                                      ├─→ Grafana
   └─ OpenTelemetry                       │
          ↓                               │
      Trace / Span                        │
          ↓                               │
      OTel Collector → Tempo ─────────────┘
```

역할만 다시 보면

```text
Application
  ├─ 로그 생성
  └─ Trace / Span 생성

Collector
  ├─ 로그 수집/전달
  └─ Telemetry 수집/전달

Loki
  └─ Log 저장/검색

Tempo
  └─ Trace 저장/검색

Prometheus
  └─ Metric 수집/저장

Grafana
  └─ Metric / Log / Trace 조회
```

로그와 Trace에 같은 `trace_id`를 넣어 연결.

```text
Prometheus
"Latency 증가"
        ↓
Tempo
"AI → DB 구간이 느림"
        ↓
Loki
"pgvector query timeout"
```
