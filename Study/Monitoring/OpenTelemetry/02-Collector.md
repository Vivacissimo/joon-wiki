---
title: 02. OpenTelemetry Collector
description: OTel Collector 수집과 전달 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, opentelemetry, collector
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. OpenTelemetry Collector

## 역할

OTel Collector = Telemetry를 받아서 가공하고 Backend로 넘기는 중간 계층.

```text
Application
   ↓
OTLP
   ↓
OTel Collector
   ├─→ Tempo
   ├─→ Loki
   └─→ 기타 Backend
```

## 기본 구조

```text
Receiver
   ↓
Processor
   ↓
Exporter
```

### Receiver

Telemetry를 받는 입력.

예시

- OTLP gRPC
- OTLP HTTP

### Processor

받은 데이터 가공.

예시

- Batch 처리
- Attribute 추가/변환
- Sampling 처리

### Exporter

가공한 데이터를 Backend로 전송.

예시

- Tempo
- Prometheus 계열 Backend
- Log Backend

## 로그 Collector와 관계

로그 수집 도구는 프로젝트 구성에 따라 다르게 잡을 수 있음.

```text
Grafana Alloy
Fluent Bit
OpenTelemetry Collector
```

Trace와 Log를 OTel 쪽으로 같이 처리할 수도 있고, 로그는 별도 Collector를 둘 수도 있음.

## 역할 구분

```text
수집/가공/전달 → OTel Collector
Trace 저장/검색 → Tempo
Log 저장/검색   → Loki
```

Collector는 저장소가 아님.

## 참고

- https://opentelemetry.io/docs/collector/

## 다음

- [01. Tempo Overview](../Tempo/01-Overview)
