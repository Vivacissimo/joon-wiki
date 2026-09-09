---
title: Trace and Span
description: Trace와 Span의 관계
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, tracing, span
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# Trace and Span

## Trace

하나의 요청 전체 흐름.

예시

```text
사용자 상품 추천 요청
        ↓
Backend
        ↓
AI
        ↓
Database
        ↓
응답 반환
```

이 전체 요청에 하나의 `trace_id`가 부여됨.

```text
trace_id = abc123
```

## Span

Trace 내부의 개별 작업 단위.

```text
Trace abc123

POST /recommend
├─ Backend Controller
├─ AI HTTP Request
└─ PostgreSQL Query
```

각 구간은 별도의 `span_id`를 가짐.

## 관계

```text
Trace = 요청 전체
Span  = 요청을 구성하는 각 작업
```

예시

```text
Trace ID: abc123

POST /recommend          3.2s
├─ Backend               100ms
├─ AI Request            2.9s
└─ DB Query              2.7s
```

이 구조를 보면 어느 구간에서 시간이 오래 걸렸는지 확인 가능.

## 로그와 연결

애플리케이션 로그에 현재 Trace와 Span 정보를 넣으면 특정 로그가 어떤 요청의 어느 작업에서 발생했는지 연결 가능.

```json
{
  "trace_id": "abc123",
  "span_id": "def456",
  "level": "ERROR",
  "message": "database timeout"
}
```

다음: [Context Propagation](/Monitoring/Tracing/03-Context-Propagation)
