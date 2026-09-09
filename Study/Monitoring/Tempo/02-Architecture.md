---
title: 02. Tempo Architecture
description: Trace 데이터 흐름과 Grafana 연결 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tempo, architecture
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. Tempo Architecture

## 기본 구조

```text
Backend ─┐
AI      ─┼─ OpenTelemetry Instrumentation
Frontend─┘
      ↓
OTel Collector
      ↓
Tempo
      ↓
Grafana
```

## 처리 순서

1. 애플리케이션에서 Trace / Span 생성
2. 서비스 간 Trace Context 전달
3. OTel Collector로 Trace 전송
4. Collector가 Tempo로 전달
5. Tempo에 Trace 저장
6. Grafana에서 조회

## 예시

```text
Trace ID: abc123

POST /api/recommend
├─ backend.controller    50ms
├─ backend.service       80ms
├─ ai.http              2.8s
└─ db.query             2.5s
```

어느 구간이 전체 시간을 많이 쓰는지 확인.

## Loki와 같이 볼 때

```text
                 Grafana
                /       \
             Loki       Tempo
             Log        Trace
              \          /
               trace_id
```

## Prometheus까지 포함

```text
                    Grafana
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Prometheus         Loki           Tempo
    Metrics           Logs           Traces
```

## 참고

- https://grafana.com/docs/tempo/latest/introduction/architecture/

## 다음

- [04. Log Trace Correlation](../Tracing/04-Log-Trace-Correlation)
