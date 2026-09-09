---
title: 01. Distributed Tracing Overview
description: 분산 추적 목적과 전체 흐름
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, observability
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 01. Distributed Tracing Overview

## 개념

한 요청이 여러 서비스나 컴포넌트를 거치는 흐름을 하나로 묶어서 보는 것.

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

로그만 보면 서비스별 기록이 따로 보임.

Trace가 있으면 같은 요청으로 묶어서 확인 가능.

## 왜 필요한가

Metric은 `느려졌다` 정도까지 확인.

Trace는 `어디에서 느려졌는지` 확인.

Log는 `왜 느려졌는지` 확인.

```text
Metric
Latency 증가
   ↓
Trace
AI 호출 구간 지연
   ↓
Log
pgvector timeout
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

## 역할

- OpenTelemetry → Trace / Span 생성 + Context 전파
- OTel Collector → Trace 수집/전달
- Tempo → Trace 저장/검색
- Grafana → Trace 확인

## 다음

- [02. Trace and Span](./02-Trace-Span)
