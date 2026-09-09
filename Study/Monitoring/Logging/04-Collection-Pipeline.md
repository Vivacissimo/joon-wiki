---
title: Log Collection Pipeline
description: Kubernetes 로그 수집 흐름과 구성 요소
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, kubernetes, loki
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# Log Collection Pipeline

## 전체 흐름

애플리케이션이 JSON 로그를 `stdout/stderr`로 출력하고, Collector가 로그를 읽어 Loki로 전달.

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

## 역할 구분

### Application

로그 이벤트 생성.

예시

```json
{
  "level": "ERROR",
  "service_name": "backend",
  "trace_id": "abc123",
  "event": "database_timeout"
}
```

### Collector

로그를 실제로 읽고 전달하는 구성 요소.

주요 역할

- Container Log 읽기
- JSON 파싱
- Kubernetes Metadata 추가
- 불필요한 필드 제거/변환
- Loki 등 저장소로 전송

예시 도구

- Grafana Alloy
- Fluent Bit
- OpenTelemetry Collector

### Loki

로그 저장 및 검색 담당.

Loki가 Pod에서 직접 로그를 긁어오는 개념으로 보기보다는 Collector가 전달한 로그를 저장하고 조회하는 Backend로 이해.

### Grafana

Loki를 Data Source로 연결해 로그를 검색하고 시각화.

## Kubernetes Metadata

Collector 단계에서 다음 정보를 로그에 추가 가능.

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

## 수집 대상과 저장소를 구분

```text
로그를 무엇으로 남길지
→ Application / Security 정책

로그를 어떻게 가져올지
→ Collector

로그를 어디에 저장할지
→ Loki

로그를 어디서 볼지
→ Grafana
```

다음: [Loki Overview](/Monitoring/Loki/01-Overview)
