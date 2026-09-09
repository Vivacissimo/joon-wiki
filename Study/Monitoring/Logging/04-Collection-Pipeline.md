---
title: 04. Log Collection Pipeline
description: Kubernetes 로그 수집 흐름과 구성 요소
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, kubernetes, loki
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 04. Log Collection Pipeline

## 전체 흐름

애플리케이션에서 JSON 로그 출력 → Collector가 읽음 → Loki에 저장 → Grafana에서 확인.

```text
Application
   ↓
stdout / stderr
   ↓
Container Log
   ↓
Collector
(Alloy / Fluent Bit / OTel Collector)
   ↓
Loki
   ↓
Grafana
```

## Application

로그를 만드는 곳.

```json
{
  "level": "ERROR",
  "service_name": "backend",
  "trace_id": "abc123",
  "event": "database_timeout"
}
```

## Collector

실제로 로그를 읽고 전달하는 쪽.

하는 일

- Container Log 읽기
- JSON 파싱
- Kubernetes Metadata 추가
- 필드 변환/제거
- Loki로 전송

도구 예시

- Grafana Alloy
- Fluent Bit
- OpenTelemetry Collector

## Loki

로그 저장 + 검색.

Loki가 Pod에 들어가서 직접 로그를 만드는 구조가 아님.

```text
Application → 로그 생성
Collector   → 로그 수집/전달
Loki        → 로그 저장/검색
Grafana     → 로그 확인
```

## Kubernetes Metadata

Collector에서 같이 붙일 수 있는 값.

```text
namespace
pod_name
container_name
node_name
labels
```

예시

```json
{
  "service_name": "backend",
  "namespace": "prod",
  "pod_name": "backend-6cc78-abcde",
  "container_name": "backend",
  "message": "database timeout"
}
```

## 다음

- [01. Loki Overview](../Loki/01-Overview)
