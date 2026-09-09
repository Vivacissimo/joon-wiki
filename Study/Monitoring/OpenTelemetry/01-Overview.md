---
title: 01. OpenTelemetry Overview
description: OpenTelemetry 역할과 Telemetry 흐름
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, opentelemetry, observability
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 01. OpenTelemetry Overview

## 역할

OpenTelemetry = Metrics, Logs, Traces 같은 Telemetry 데이터를 만들고 전달하기 위한 표준 + 도구.

분산 추적에서는 애플리케이션에 계측을 붙여 Trace / Span 생성 + Trace Context 전파.

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

## OpenTelemetry와 Tempo 차이

```text
OpenTelemetry
→ Trace 생성 / 전파 / 전달

Tempo
→ Trace 저장 / 검색
```

Tempo가 애플리케이션 요청을 보고 Trace를 직접 만들어주는 건 아님.

## 구성

### Instrumentation

어디를 Trace로 볼지 계측.

예시

- HTTP Server 요청
- HTTP Client 요청
- Database 호출
- 외부 API 호출

### SDK / Agent

언어별 SDK나 자동 계측 Agent 사용.

Trace / Span 생성 + Context 전파 처리.

### Collector

애플리케이션에서 받은 Telemetry를 Backend로 전달.

## 로그와 연결

현재 Span의 `trace_id`, `span_id`를 로그에도 넣음.

```text
Loki Log
trace_id=abc123
      ↕
OpenTelemetry Trace
trace_id=abc123
```

## 참고

- https://opentelemetry.io/docs/
- https://opentelemetry.io/docs/concepts/signals/traces/

## 다음

- [02. OpenTelemetry Collector](./02-Collector)
