---
title: 03. Context Propagation
description: 서비스 간 Trace Context 전달 방식
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, opentelemetry
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 03. Context Propagation

## 개념

하나의 요청이 여러 서비스를 이동할 때 같은 Trace Context를 다음 서비스로 전달하는 과정.

```text
Frontend
trace_id=abc123
   ↓
Backend
trace_id=abc123
   ↓
AI
trace_id=abc123
```

서비스마다 다른 Trace ID를 생성하면 하나의 요청 흐름으로 연결할 수 없음.

## 전달되는 주요 값

- Trace ID
- Span ID
- Trace Flags
- Trace State

일반적으로 OpenTelemetry가 HTTP 요청 헤더를 통해 Context를 전파.

## W3C Trace Context

대표적인 전파 형식은 `traceparent` 헤더.

```text
traceparent: 00-<trace-id>-<parent-span-id>-01
```

애플리케이션 코드에서 직접 Trace ID 문자열을 임의로 전달하기보다 OpenTelemetry Instrumentation을 통해 자동 전파하는 방식을 우선 고려.

## 왜 필요한가

Context Propagation이 정상이어야 다음과 같은 전체 Trace가 생성됨.

```text
POST /recommend
├─ Backend
│  └─ AI Request
│     └─ Database Query
```

전파가 끊기면 서로 별개의 Trace로 보임.

```text
Trace A → Backend
Trace B → AI
Trace C → DB
```

## 검증 포인트

- Backend에서 AI 호출 시 같은 Trace ID가 유지되는지
- 비동기 작업에서도 Trace Context가 유지되는지
- 로그에 기록된 trace_id와 Tempo의 Trace ID가 일치하는지

## 참고

- OpenTelemetry Context Propagation: https://opentelemetry.io/docs/concepts/context-propagation/

다음: [04. Log Trace Correlation](/Monitoring/Tracing/04-Log-Trace-Correlation)
