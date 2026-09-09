---
title: 03. Structured JSON Log
description: 공통 JSON 로그 포맷과 필드 구성
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, json
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 03. Structured JSON Log

## 왜 JSON으로 남기나

문자열 로그

```text
2026-09-09 ERROR PaymentService payment failed
```

사람이 보기엔 쉽지만 필드 단위 검색이 불편.

JSON 로그

```json
{
  "timestamp": "2026-09-09T10:31:15Z",
  "level": "ERROR",
  "service_name": "backend",
  "environment": "prod",
  "event": "demand_create_failed",
  "http_method": "POST",
  "http_path": "/api/demands",
  "status_code": 500,
  "duration_ms": 1832,
  "trace_id": "abc123",
  "span_id": "def456",
  "error_type": "DatabaseTimeout",
  "message": "demand creation failed"
}
```

`service_name`, `status_code`, `trace_id` 같은 필드로 검색 가능.

## 공통 필드

| 필드 | 내용 |
|---|---|
| `timestamp` | 발생 시각 |
| `level` | INFO / WARN / ERROR |
| `service_name` | backend / ai / frontend |
| `environment` | dev / prod |
| `event` | 이벤트 이름 |
| `trace_id` | 요청 전체 식별 |
| `span_id` | Trace 안의 작업 식별 |
| `request_id` | HTTP 요청 식별 |
| `http_method` | GET / POST 등 |
| `http_path` | API 경로 |
| `status_code` | HTTP 응답 코드 |
| `duration_ms` | 처리 시간 |
| `error_type` | 에러 종류 |
| `message` | 로그 내용 |

## event 이름

자유 문장보다 같은 상황에 같은 이름 사용.

```text
login_success
login_failed
demand_created
demand_create_failed
seller_condition_changed
group_buy_closed
external_api_timeout
database_timeout
```

## Log Level

### INFO

정상 흐름.

```text
demand_created
login_success
recommendation_completed
```

### WARN

실패 전 이상 징후.

```text
slow_response
retry_occurred
external_api_delay
```

### ERROR

실제 실패.

```text
database_timeout
http_500
external_api_failed
unhandled_exception
```

## Trace 연결

로그에 `trace_id`, `span_id` 포함.

```text
JSON Log
trace_id=abc123
      ↕
Tempo Trace
trace_id=abc123
```

같은 `trace_id`로 Log ↔ Trace 연결.

## 다음

- [04. Log Collection Pipeline](./04-Collection-Pipeline)
