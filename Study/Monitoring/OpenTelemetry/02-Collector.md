---
title: 02. OpenTelemetry Collector
description: OTel Collector의 수집과 전달 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, opentelemetry, collector
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. OpenTelemetry Collector

## 역할

OTel Collector는 애플리케이션이나 Agent에서 전달된 Telemetry를 받아 가공하고 원하는 Backend로 전달하는 중간 계층.

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

## 기본 구성

Collector Pipeline은 크게 세 부분으로 구성.

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

받은 데이터를 가공.

예시

- Batch 처리
- Attribute 추가/변환
- Sampling 관련 처리

### Exporter

처리한 데이터를 외부 Backend로 전송.

예시

- Tempo
- Prometheus 계열 Backend
- Log Backend

## 로그 Collector와의 관계

로그 수집은 Grafana Alloy, Fluent Bit, OTel Collector 중 프로젝트 목적에 맞는 도구를 선택 가능.

OpenTelemetry 기반으로 Trace와 Log 처리를 통합하면 구성 요소를 줄일 수 있지만, 실제 선택은 운영 요구사항과 Helm Chart 지원 범위를 보고 결정.

## 주의

Collector는 Trace를 장기 저장하는 DB가 아님.

```text
수집/가공/전달 → OTel Collector
Trace 저장/검색 → Tempo
Log 저장/검색   → Loki
```

## 참고

- OpenTelemetry Collector: https://opentelemetry.io/docs/collector/
