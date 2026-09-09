---
title: Kiali
description: Istio Service Mesh 트래픽 시각화
published: true
date: 2026-09-09T10:57:00.000Z
tags: monitoring, kiali, istio, service-mesh
editor: markdown
dateCreated: 2026-06-04T07:49:43.020Z
---

# Kiali

## 역할

Istio Service Mesh 상태와 서비스 간 트래픽을 화면으로 보는 도구.

```text
Frontend
   ↓
Backend
   ↓
AI
```

이런 호출 관계를 Service Graph 형태로 확인.

## 주로 보는 것

- 서비스 간 호출 관계
- Request Rate
- Error Rate
- Response Time
- HTTP 상태
- mTLS 상태
- Istio Traffic Routing 상태

## Kiali가 직접 저장하는 건 아님

Kiali 자체를 Metric / Log / Trace 저장소로 보는 건 아님.

```text
Prometheus
   ↓
Metric
   ↓
Kiali에서 시각화
```

Observability 도구 역할을 나누면

```text
Prometheus → Metric 저장/조회
Loki       → Log 저장/검색
Tempo      → Trace 저장/검색
Grafana    → Metric / Log / Trace 화면
Kiali      → Istio 서비스 관계 / 트래픽 화면
```

## Grafana와 차이

Grafana는 Metric, Log, Trace 데이터를 대시보드로 보는 쪽.

Kiali는 Istio 환경에서 서비스끼리 어떻게 연결되고 트래픽이 어떻게 흐르는지 보는 쪽.

```text
Grafana
→ 수치 / 로그 / Trace 중심

Kiali
→ 서비스 관계 / Mesh Traffic 중심
```

## 같이 보기

- [Monitoring](../../Monitoring)
- [Grafana Install](../Grafana/Install)
