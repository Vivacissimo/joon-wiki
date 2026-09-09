---
title: Loki Overview
description: Loki의 역할과 로그 저장 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, loki, logging
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# Loki Overview

## 역할

Loki는 로그를 저장하고 검색하는 Log Backend.

```text
Application
   ↓
Collector
   ↓
Loki
   ↓
Grafana
```

Collector가 수집한 로그를 Loki로 전달하고 Grafana에서 Loki를 Data Source로 연결해 조회.

## Loki가 하는 일

- 로그 저장
- Label 기준 로그 검색
- 시간 범위별 조회
- Grafana 연동
- Trace ID 기반 로그 검색

## Loki가 하지 않는 일

Loki 자체를 애플리케이션 로그 생성기나 분산 추적 도구로 보지 않음.

```text
로그 생성 → Application
로그 수집/전송 → Collector
로그 저장/검색 → Loki
로그 시각화 → Grafana
```

## Label

Loki에서는 로그를 구분하기 위한 Metadata를 Label로 사용.

예시

```text
service_name="backend"
environment="prod"
namespace="moongcheap"
```

로그 본문에는 상세 이벤트와 trace_id 등이 포함.

```json
{
  "event": "demand_create_failed",
  "trace_id": "abc123",
  "status_code": 500,
  "message": "database timeout"
}
```

## 참고

- Grafana Loki: https://grafana.com/docs/loki/latest/

다음: [Loki Architecture](/Monitoring/Loki/02-Architecture)
