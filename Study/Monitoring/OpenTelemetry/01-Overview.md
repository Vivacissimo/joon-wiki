---
title: OpenTelemetry Overview
description: OpenTelemetry의 역할과 Telemetry 데이터 흐름
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, opentelemetry, observability
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# OpenTelemetry Overview

## 역할

OpenTelemetry는 Metrics, Logs, Traces 같은 Telemetry 데이터를 생성하고 수집하고 전달하기 위한 표준과 도구 모음.

분산 추적에서는 애플리케이션에 Instrumentation을 적용해 Trace와 Span을 만들고 서비스 간 Trace Context를 전파하는 데 사용.

```text
Application
   ↓
OpenTelemetry Instrumentation
   ↓
Trace / Span 생성
   ↓
OTel Collector
   ↓
Tempo
```

## OpenTelemetry와 Tempo의 차이

```text
OpenTelemetry
→ Trace를 생성하고 전달하는 쪽

Tempo
→ 생성된 Trace를 저장하고 조회하는 쪽
```

Tempo가 애플리케이션 요청을 자동으로 분석해 Trace를 만드는 것은 아님.

## 주요 구성

### Instrumentation

애플리케이션 코드나 Framework에 계측 적용.

예시 대상

- HTTP Server 요청
- HTTP Client 요청
- Database 호출
- 외부 API 호출

### SDK / Agent

언어별 OpenTelemetry SDK 또는 자동 계측 Agent를 통해 Span 생성과 Context Propagation 처리.

### Collector

애플리케이션에서 전달된 Telemetry를 받아 Backend로 전달.

## 로그 연계

현재 실행 중인 Span의 `trace_id`, `span_id`를 로그에 포함하면 Loki의 로그와 Tempo의 Trace를 연결 가능.

```text
Application Log
trace_id=abc123
        ↕
OpenTelemetry Trace
trace_id=abc123
```

## 참고

- OpenTelemetry: https://opentelemetry.io/docs/
- Traces: https://opentelemetry.io/docs/concepts/signals/traces/

다음: [OpenTelemetry Collector](/Monitoring/OpenTelemetry/02-Collector)
