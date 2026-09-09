---
title: 02. Tempo Architecture
description: Trace 데이터 흐름과 Grafana 연동 구조
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

## 처리 단계

1. 애플리케이션에서 Trace와 Span 생성
2. 서비스 간 Trace Context 전파
3. OTel Collector로 Trace 전송
4. Collector가 Trace 데이터를 Tempo로 Export
5. Tempo가 Trace 저장
6. Grafana에서 Tempo Data Source로 조회

## 예시

```text
Trace ID: abc123

POST /api/recommend
├─ backend.controller    50ms
├─ backend.service       80ms
├─ ai.http              2.8s
└─ db.query             2.5s
```

이 구조를 통해 어느 구간이 전체 응답 시간을 차지하는지 확인.

## Loki와 함께 사용할 때

```text
                 Grafana
                /       \
             Loki       Tempo
             Log        Trace
              \          /
               trace_id
```

공통 Trace ID를 기준으로 로그와 Trace를 연결.

## Prometheus까지 포함한 전체 구조

```text
                    Grafana
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Prometheus         Loki           Tempo
    Metrics           Logs           Traces
```

## 참고

- Tempo Architecture: https://grafana.com/docs/tempo/latest/introduction/architecture/
