---
title: 02. Loki Architecture
description: Collector와 Loki, Grafana 연결 구조
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, loki, architecture
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. Loki Architecture

## 기본 구조

```text
Backend Pod ─┐
AI Pod      ─┼─ stdout/stderr
Frontend Pod─┘
      ↓
Collector
      ↓
Loki
      ↓
Grafana
```

## 처리 순서

1. 애플리케이션에서 JSON 로그 출력
2. Kubernetes Node에 Container Log 생성
3. Collector가 로그 읽기
4. JSON 파싱 + Kubernetes Metadata 추가
5. Loki로 전송
6. Grafana에서 조회

## 로그 찾는 기준

```text
service_name = backend
level = ERROR
trace_id = abc123
```

## Tempo와 연결

Loki 로그와 Tempo Trace에 같은 `trace_id` 사용.

```text
Loki
ERROR database timeout
trace_id=abc123
      ↓
Tempo
Trace abc123
```

## 데이터별 저장소

| 데이터 | 저장소 |
|---|---|
| Metric | Prometheus |
| Log | Loki |
| Trace | Tempo |

## 참고

- https://grafana.com/docs/loki/latest/get-started/architecture/

## 다음

- [01. Distributed Tracing Overview](../Tracing/01-Overview)
