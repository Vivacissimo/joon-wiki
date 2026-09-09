---
title: 01. Logging Overview
description: 운영 로그의 목적과 수집 범위
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, observability
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 01. Logging Overview

## 로그를 남기는 이유

로그는 시스템에서 발생한 사건을 시간 순서대로 기록한 데이터.

주요 목적

- API 요청 성공/실패 확인
- 애플리케이션 오류 원인 확인
- 인증/권한 변경 이력 확인
- 비정상 요청 및 보안 이벤트 추적
- 특정 사용자 요청의 처리 흐름 확인
- 장애 발생 시 원인 분석

## 무엇을 로그로 남길지 결정하는 기준

무조건 모든 데이터를 남기는 방식이 아니라, 운영 중 답해야 하는 질문에서 시작.

```text
운영 중 확인해야 하는 질문
        ↓
장애/보안 시나리오
        ↓
남겨야 할 이벤트
        ↓
이벤트에 필요한 필드
        ↓
로그 포맷 정의
```

예시

> 상품 추천 요청이 왜 실패했는가?

필요한 정보

- 어떤 서비스에서 발생했는지
- 어떤 API였는지
- HTTP 상태 코드
- 처리 시간
- 오류 종류
- 요청 전체를 식별할 trace_id

## 로그의 주요 종류

| 종류 | 예시 | 목적 |
|---|---|---|
| Access Log | `POST /api/demands 201 120ms` | 요청/응답 확인 |
| Application Log | 수요 등록 성공 | 서비스 동작 확인 |
| Error Log | DB timeout | 장애 원인 확인 |
| Security Log | 로그인 실패 | 공격/이상 행위 탐지 |
| Audit Log | 셀러 조건 변경 | 변경 이력 추적 |
| External Call Log | AI API timeout | 서비스 간 호출 문제 확인 |

## 주의

로그에는 비밀번호, Access Token, Refresh Token, API Key, 카드번호 등 민감정보 원문을 기록하지 않음.

민감정보 자체보다 `누가`, `언제`, `어떤 리소스에`, `어떤 작업을 했는지`를 기록.

## 다음

- [02. Log Data Dictionary](/Monitoring/Logging/02-Log-Data-Dictionary)
- [03. Structured JSON Log](/Monitoring/Logging/03-JSON-Log-Format)
- [04. Log Collection Pipeline](/Monitoring/Logging/04-Collection-Pipeline)
