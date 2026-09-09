---
title: 02. Trace and Span
description: Trace와 Span 관계
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, span
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. Trace and Span

## Trace

요청 하나의 전체 흐름.

```text
사용자 상품 추천 요청
        ↓
Backend
        ↓
AI
        ↓
Database
        ↓
응답
```

이 전체 요청에 하나의 `trace_id` 사용.

```text
trace_id = abc123
```

## Span

Trace 안의 작업 단위.

```text
Trace abc123

POST /recommend
├─ Backend Controller
├─ AI HTTP Request
└─ PostgreSQL Query
```

각 Span은 자기 `span_id`를 가짐.

## 차이

```text
Trace = 요청 전체
Span  = 요청 안의 각 작업
```

예시

```text
Trace ID: abc123

POST /recommend          3.2s
├─ Backend               100ms
├─ AI Request            2.9s
└─ DB Query              2.7s
```

이걸 보고 어느 구간에서 시간이 많이 걸렸는지 확인.

## 로그와 연결

로그에도 현재 Trace / Span 값 기록.

```json
{
  "trace_id": "abc123",
  "span_id": "def456",
  "level": "ERROR",
  "message": "database timeout"
}
```

## 다음

- [03. Context Propagation](./03-Context-Propagation)
