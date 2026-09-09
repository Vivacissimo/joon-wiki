---
title: 01. Jaeger Overview
description: 분산 추적용 Trace 저장과 조회
published: true
date: 2026-09-09T12:04:00.000Z
tags: monitoring, jaeger, tracing, opentelemetry
editor: markdown
dateCreated: 2026-09-09T12:04:00.000Z
---

# 01. Jaeger Overview

## Jaeger

분산 추적 데이터를 저장하고 조회하는 Trace Backend.

하나의 요청이 여러 서비스를 거칠 때 전체 흐름을 Trace로 묶어서 확인.

```text
User Request
    ↓
Frontend
    ↓
Backend
    ↓
AI
    ↓
DB
```

Trace 화면에서는 이런 식으로 보임.

```text
Trace ID: abc123

POST /recommend
├─ Backend        120ms
├─ AI Request     2.4s
└─ DB Query       2.1s
```

어느 구간이 오래 걸렸는지, 어느 Span에서 실패했는지 확인.

## Trace 데이터 생성

Jaeger가 애플리케이션 요청을 보고 자동으로 Trace를 만드는 건 아님.

애플리케이션 쪽에서 OpenTelemetry로 Trace / Span 생성.

```text
Application
    ↓
OpenTelemetry
    ↓
Trace / Span
    ↓
OTel Collector
    ↓
Jaeger
```

## Tempo와 관계

Jaeger와 Tempo는 둘 다 Trace 영역.

```text
Tempo  → Trace 저장/검색 + Grafana에서 확인
Jaeger → Trace 저장/조회 + Jaeger UI 사용 가능
```

둘을 같이 쓸 필요는 없음.

현재 Prometheus + Loki + Grafana 흐름에서는 Tempo 쪽이 더 단순하게 붙음.

Jaeger를 쓰면 Jaeger UI에서 Trace를 바로 볼 수 있다는 차이가 있음.

## Kiali와 차이

```text
Kiali
→ 서비스끼리 어떻게 연결됐는지
→ Istio 트래픽 상태

Jaeger
→ 요청 하나가 어떤 서비스를 거쳤는지
→ Span별 처리 시간
```

## 다음

- [02. Jaeger Architecture](./02-Architecture)
