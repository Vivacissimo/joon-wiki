---
title: 02. Log Data Dictionary
description: 필수 로그 이벤트와 필드 정리
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, security
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. Log Data Dictionary

## 개념

`어떤 상황에서 어떤 값을 로그로 남길지` 정리한 기준표.

보안팀에서 만든 항목은 침해사고 대응, 감사, 이상 행위 확인에 필요한 이벤트와 필드 중심.

예시

| 이벤트 | 필요한 값 |
|---|---|
| 로그인 | 사용자 ID, 성공/실패, IP, User-Agent, 시각 |
| 권한 변경 | 대상 리소스, 변경 전/후 권한, 관리자 액션 |
| 수요 등록 | 사용자 ID, 상품 카테고리, 목표 수량, 마감일, IP |
| 셀러 조건 변경 | 셀러 ID, 수요 ID, 변경 전/후 값, 변경 시각 |
| 공동구매 마감 | 수요 ID, 마감 유형, 참여 인원, 마감 시각 |

## Dictionary → JSON 필드

문서에 적힌 한글 항목을 그대로 JSON Key로 쓰는 건 아님.

실제 로그에서는 필드명을 정해서 사용.

| Dictionary | JSON Key 예시 |
|---|---|
| 사용자 ID | `user_id` |
| 성공/실패 | `result` |
| IP | `client_ip` |
| User-Agent | `user_agent` |
| 시각 | `timestamp` |
| 응답 코드 | `status_code` |

## 공통 필드 + 이벤트별 필드

```text
공통
├─ timestamp
├─ level
├─ service_name
├─ environment
├─ event
├─ trace_id
└─ span_id

이벤트별
├─ login
│  └─ user_id, result, client_ip
├─ demand_created
│  └─ product_category, target_quantity
└─ seller_condition_changed
   └─ seller_id, before, after
```

## 민감정보

`개인정보 접근 로그` = 개인정보 원문 저장이 아님.

남기면 안 되는 형태

```json
{
  "card_number": "1234-5678-9012-3456"
}
```

남기는 형태

```json
{
  "event": "personal_data_access",
  "user_id": "user-128",
  "resource_type": "payment_info",
  "action": "READ",
  "result": "SUCCESS"
}
```

## 흐름

```text
Log Data Dictionary
        ↓
남길 이벤트 / 값 결정
        ↓
실제 JSON 필드명 결정
        ↓
애플리케이션 로그에 적용
```

## 다음

- [03. Structured JSON Log](./03-JSON-Log-Format)
