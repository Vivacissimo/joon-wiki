---
title: Structured JSON Log
description: 공통 JSON 로그 포맷과 필드 구성
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, json
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# Structured JSON Log

## 왜 JSON으로 남기는가

문자열 로그는 사람이 읽기에는 쉽지만 필드별 검색과 집계가 불편.

```text
2026-09-09 ERROR PaymentService payment failed
```

JSON 형태로 구조화하면 `service_name`, `status_code`, `trace_id` 같은 값을 기준으로 검색 가능.

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

## 권장 공통 필드

| 필드 | 목적 |
|---|---|
| `timestamp` | 이벤트 발생 시각 |
| `level` | INFO / WARN / ERROR |
| `service_name` | backend / ai / frontend |
| `environment` | dev / prod |
| `event` | 이벤트 식별 이름 |
| `trace_id` | 요청 전체 식별 |
| `span_id` | Trace 내부 작업 식별 |
| `request_id` | HTTP 요청 식별이 필요한 경우 |
| `http_method` | GET / POST 등 |
| `http_path` | API 경로 |
| `status_code` | HTTP 응답 코드 |
| `duration_ms` | 처리 시간 |
| `error_type` | 오류 종류 |
| `message` | 사람이 읽는 설명 |

## 이벤트 이름 예시

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

이벤트 이름은 자유 문장보다 고정된 식별값 사용을 권장.

## Log Level 기준

### INFO

정상적인 주요 흐름.

```text
demand_created
login_success
recommendation_completed
```

### WARN

실패는 아니지만 이상 징후가 있는 상황.

```text
slow_response
retry_occurred
external_api_delay
```

### ERROR

요청이나 작업이 실제로 실패한 상황.

```text
database_timeout
http_500
external_api_failed
unhandled_exception
```

## Trace 연계

분산 추적을 사용할 경우 로그에 `trace_id`, `span_id`를 포함.

```text
JSON Log trace_id=abc123
        ↕
Tempo Trace ID=abc123
```

동일한 Trace ID를 통해 Grafana에서 로그와 Trace를 연결.

다음: [Log Collection Pipeline](/Monitoring/Logging/04-Collection-Pipeline)
