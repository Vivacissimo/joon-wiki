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

## 로그

시스템에서 발생한 일을 시간 순서대로 남긴 기록.

주로 보는 것

- API 요청 성공/실패
- 애플리케이션 오류
- 로그인/인증 실패
- 권한 변경
- 서비스 간 호출 실패
- 특정 요청의 처리 흐름
- 장애 발생 원인

## 뭘 남길지 정하는 순서

모든 값을 무조건 남기는 건 아님.

```text
운영 중 확인하고 싶은 것
        ↓
장애 / 보안 상황
        ↓
남겨야 하는 이벤트
        ↓
필요한 필드
        ↓
로그 형식
```

예시

```text
상품 추천 요청이 왜 실패했지?
        ↓
어느 서비스?
어느 API?
상태 코드?
처리 시간?
에러 종류?
같은 요청의 trace_id?
```

## 로그 종류

| 종류 | 예시 | 보는 이유 |
|---|---|---|
| Access Log | `POST /api/demands 201 120ms` | 요청/응답 확인 |
| Application Log | 수요 등록 성공 | 서비스 흐름 확인 |
| Error Log | DB timeout | 장애 원인 확인 |
| Security Log | 로그인 실패 | 이상 행위 확인 |
| Audit Log | 셀러 조건 변경 | 변경 이력 확인 |
| External Call Log | AI API timeout | 서비스 간 호출 확인 |

## 남기면 안 되는 값

- 비밀번호
- Access Token / Refresh Token
- API Key
- 카드번호
- 민감정보 원문

민감정보 내용 자체보다 `누가`, `언제`, `무엇을`, `어떻게 처리했는지` 기록.

## 다음

- [02. Log Data Dictionary](./02-Log-Data-Dictionary)
