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

한 요청이 다음 서비스로 넘어갈 때 Trace 정보를 같이 전달하는 것.

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

서비스마다 새 Trace ID를 만들면 요청 하나로 연결되지 않음.

## 같이 전달되는 값

- Trace ID
- Span ID
- Trace Flags
- Trace State

HTTP에서는 Trace Context를 Header로 전달.

## W3C Trace Context

대표 Header

```text
traceparent: 00-<trace-id>-<parent-span-id>-01
```

Trace ID 문자열을 애플리케이션에서 직접 만들어 넘기기보다 OpenTelemetry Instrumentation에서 처리.

## 연결됐을 때

```text
POST /recommend
├─ Backend
│  └─ AI Request
│     └─ Database Query
```

Context가 끊기면

```text
Trace A → Backend
Trace B → AI
Trace C → DB
```

서로 다른 요청처럼 보임.

## 확인할 것

- Backend → AI 호출에서 Trace가 이어지는지
- 비동기 작업에서도 Context가 이어지는지
- 로그의 `trace_id`와 Tempo Trace ID가 같은지

## 참고

- https://opentelemetry.io/docs/concepts/context-propagation/

## 다음

- [01. OpenTelemetry Overview](../OpenTelemetry/01-Overview)
