---
title: 02. Kiali Architecture
description: Kiali가 Istio와 Prometheus 데이터를 보는 흐름
published: true
date: 2026-09-09T12:04:00.000Z
tags: monitoring, kiali, istio, prometheus
editor: markdown
dateCreated: 2026-09-09T12:04:00.000Z
---

# 02. Kiali Architecture

## 흐름

```text
User Request
    ↓
Istio Envoy Sidecar
    ↓
Service A ─────────→ Service B
    │                  │
    └──── Metric ──────┘
            ↓
       Prometheus
            ↓
          Kiali
            ↓
 Service Graph / Traffic
```

Kiali가 애플리케이션 로그를 읽어서 서비스 관계를 만드는 구조는 아님.

Istio가 만든 Telemetry + Prometheus Metric을 이용해서 서비스 관계와 트래픽 상태를 보여줌.

## Istio와 관계

```text
Kubernetes
   ↓
Istio
   ├─ Envoy Sidecar
   ├─ VirtualService
   ├─ DestinationRule
   └─ mTLS
        ↓
      Kiali
```

Kiali에서 Istio 설정과 실제 트래픽 상태를 같이 확인.

## Canary 볼 때

예를 들어 Backend v1 / v2로 트래픽 분리.

```text
Frontend
   ↓
Backend
   ├─ v1 80%
   └─ v2 20%
```

Kiali에서 확인할 부분

- v1 / v2 요청 비율
- 에러가 어느 버전에 몰리는지
- 응답시간 차이
- 트래픽 라우팅이 설정대로 들어가는지

## 장애 볼 때

```text
Kiali
"어느 서비스 구간에서 에러가 늘었나?"
        ↓
Grafana / Prometheus
"에러율과 지연시간이 얼마나 늘었나?"
        ↓
Tempo / Jaeger
"요청이 어느 Span에서 느려졌나?"
        ↓
Loki
"실제 에러 메시지가 뭔가?"
```

## 앞 문서

- [01. Kiali Overview](./01-Overview)
