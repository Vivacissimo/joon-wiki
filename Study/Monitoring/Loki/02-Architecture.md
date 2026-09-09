---
title: 02. Loki Architecture
description: Collector와 Loki, Grafana의 연결 구조
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

## 처리 단계

1. 애플리케이션이 stdout/stderr에 JSON 로그 출력
2. Kubernetes Node에 Container Log 생성
3. Collector가 로그 파일을 읽음
4. JSON 파싱 및 Kubernetes Metadata 추가
5. Loki로 전달
6. Grafana에서 Loki Data Source로 조회

## 예시 검색 기준

```text
service_name = backend
level = ERROR
trace_id = abc123
```

## 분산 추적과 연결

Loki 로그에 `trace_id`가 포함되어 있고 Tempo에도 동일한 Trace ID가 존재하면 Grafana에서 Log → Trace 연결 가능.

```text
Loki
ERROR database timeout
trace_id=abc123
      ↓
Tempo
Trace abc123
```

## 저장소 역할 구분

| 데이터 | 저장소 |
|---|---|
| Metric | Prometheus |
| Log | Loki |
| Trace | Tempo |

## 참고

- Loki Architecture: https://grafana.com/docs/loki/latest/get-started/architecture/
