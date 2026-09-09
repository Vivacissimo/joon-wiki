---
title: 01. Distributed Tracing Overview
description: 분산 추적의 목적과 전체 흐름
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, observability
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 01. Distributed Tracing Overview

## 개념

분산 추적은 하나의 요청이 여러 서비스나 컴포넌트를 거치는 전체 흐름을 하나로 연결해서 보는 방식.

```text
User
 ↓
Frontend
 ↓
Backend
 ↓
AI
 ↓
Database
```

로그만 보면 각 서비스의 기록이 따로 존재하지만 Trace를 사용하면 하나의 요청으로 묶어서 확인 가능.

## 필요한 이유

메트릭으로 `느려졌다`는 사실은 알 수 있지만 어디에서 느려졌는지는 바로 알기 어려움.

```text
Metric
"Latency 증가"
   ↓
Trace
"AI 호출 구간에서 지연"
   ↓
Log
"pgvector timeout"
```

## 전체 흐름

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
   ↓
Grafana
```

## 역할 구분

- OpenTelemetry: Trace/Span 생성과 전파
- OTel Collector: Telemetry 수집 및 전달
- Tempo: Trace 저장 및 검색
- Grafana: Trace 시각화

## 다음

- [02. Trace and Span](/Monitoring/Tracing/02-Trace-Span)
- [03. Context Propagation](/Monitoring/Tracing/03-Context-Propagation)
- [04. Log Trace Correlation](/Monitoring/Tracing/04-Log-Trace-Correlation)
