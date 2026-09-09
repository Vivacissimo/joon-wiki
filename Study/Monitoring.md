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

시스템 상태를 확인하고 장애 원인을 추적하기 위한 Metrics, Logs, Traces와 관련 도구 정리.

```text
                    Grafana
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Prometheus         Loki           Tempo
    Metrics           Logs           Traces
```

각 데이터의 역할

| 데이터 | 확인하는 내용 | 대표 도구 |
|---|---|---|
| Metrics | 얼마나 느린지, 에러가 얼마나 발생하는지 | Prometheus |
| Logs | 어떤 오류가 발생했고 왜 실패했는지 | Loki |
| Traces | 요청이 어디를 거쳤고 어느 구간이 느린지 | Tempo |
| Visualization | Metric / Log / Trace 조회 및 연결 | Grafana |

장애 분석은 다음 흐름으로 연결.

```text
Metric
"문제가 발생했는가?"
   ↓
Trace
"어디에서 발생했는가?"
   ↓
Log
"왜 발생했는가?"
```

---

# 문서 구조

## Logging

애플리케이션에서 어떤 이벤트를 어떤 필드로 기록하고, 수집 파이프라인으로 전달하는지 정리.

1. [Logging Overview](/Monitoring/Logging/01-Overview)
2. [Log Data Dictionary](/Monitoring/Logging/02-Log-Data-Dictionary)
3. [Structured JSON Log](/Monitoring/Logging/03-JSON-Log-Format)
4. [Log Collection Pipeline](/Monitoring/Logging/04-Collection-Pipeline)

## Loki

로그 저장 및 검색 Backend.

1. [Loki Overview](/Monitoring/Loki/01-Overview)
2. [Loki Architecture](/Monitoring/Loki/02-Architecture)

## Distributed Tracing

하나의 요청을 Trace와 Span으로 연결해 서비스 간 처리 흐름을 추적.

1. [Distributed Tracing Overview](/Monitoring/Tracing/01-Overview)
2. [Trace and Span](/Monitoring/Tracing/02-Trace-Span)
3. [Context Propagation](/Monitoring/Tracing/03-Context-Propagation)
4. [Log Trace Correlation](/Monitoring/Tracing/04-Log-Trace-Correlation)

## OpenTelemetry

Trace 생성, Context 전파, Telemetry 수집/전달.

1. [OpenTelemetry Overview](/Monitoring/OpenTelemetry/01-Overview)
2. [OpenTelemetry Collector](/Monitoring/OpenTelemetry/02-Collector)

## Tempo

분산 추적 데이터 저장 및 검색 Backend.

1. [Tempo Overview](/Monitoring/Tempo/01-Overview)
2. [Tempo Architecture](/Monitoring/Tempo/02-Architecture)

## Grafana

Prometheus, Loki, Tempo 등의 Data Source를 연결해 Metrics, Logs, Traces를 한 화면에서 조회.

- [Grafana](/Monitoring/Grafana/Install)

## Kiali

Istio Service Mesh 환경의 서비스 토폴로지와 트래픽 흐름 확인.

- [Kiali](/Monitoring/kiali/what-is-kiali)

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

로그와 Trace에는 동일한 `trace_id`를 포함해 Grafana에서 서로 연결할 수 있도록 구성.
