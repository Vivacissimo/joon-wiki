---
title: 01. Kiali Overview
description: Istio Service Mesh 상태와 트래픽 확인
published: true
date: 2026-09-09T12:04:00.000Z
tags: monitoring, kiali, istio, service-mesh
editor: markdown
dateCreated: 2026-09-09T12:04:00.000Z
---

# 01. Kiali Overview

## Kiali

Istio Service Mesh 상태와 서비스 간 트래픽을 화면으로 보는 도구.

```text
Frontend
   ↓
Backend
   ↓
AI
```

서비스가 어떻게 연결되어 있고 트래픽이 어디로 흐르는지 Service Graph 형태로 확인.

## 보는 것

- 서비스 간 호출 관계
- Request Rate
- Error Rate
- Response Time
- HTTP 상태
- mTLS 상태
- Istio Traffic Routing 상태
- Canary / Traffic Split 상태

## 역할

```text
Istio
  ↓
서비스 간 트래픽 정보
  ↓
Prometheus Metric
  ↓
Kiali
  ↓
Service Graph / Traffic 화면
```

Kiali 자체가 Metric, Log, Trace 저장소는 아님.

```text
Prometheus → Metric 저장/조회
Loki       → Log 저장/검색
Tempo      → Trace 저장/검색
Jaeger     → Trace 저장/조회
Grafana    → Metric / Log / Trace 화면
Kiali      → Istio 서비스 관계 / 트래픽 화면
```

## Grafana와 차이

```text
Grafana
→ Metric / Log / Trace 중심

Kiali
→ Service Mesh / 서비스 관계 / 트래픽 흐름 중심
```

Istio를 쓰지 않으면 Kiali를 굳이 넣을 이유가 적음.

## 다음

- [02. Kiali Architecture](./02-Architecture)
