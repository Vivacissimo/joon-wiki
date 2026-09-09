---
title: 01. Loki Overview
description: Loki 역할과 로그 저장 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, loki, logging
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 01. Loki Overview

## 역할

Loki = 로그 저장 + 검색 Backend.

```text
Application
   ↓
Collector
   ↓
Loki
   ↓
Grafana
```

Collector가 가져온 로그를 Loki에 저장하고 Grafana에서 조회.

## 하는 일

- 로그 저장
- Label 기준 검색
- 시간 범위 검색
- Grafana 연결
- `trace_id` 기준 로그 찾기

## 역할 구분

```text
로그 생성       → Application
로그 수집/전달  → Collector
로그 저장/검색  → Loki
로그 확인       → Grafana
```

## Label

로그를 묶어서 찾을 때 쓰는 Metadata.

```text
service_name="backend"
environment="prod"
namespace="moongcheap"
```

로그 본문에는 실제 이벤트 내용과 `trace_id` 등이 들어감.

```json
{
  "event": "demand_create_failed",
  "trace_id": "abc123",
  "status_code": 500,
  "message": "database timeout"
}
```

## 참고

- https://grafana.com/docs/loki/latest/

## 다음

- [02. Loki Architecture](./02-Architecture)
