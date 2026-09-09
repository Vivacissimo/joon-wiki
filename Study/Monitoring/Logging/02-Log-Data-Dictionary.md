---
title: 02. Log Data Dictionary
description: 필수 로그 이벤트와 필드를 정의하는 기준
published: true
date: 2026-09-09T10:00:00.000Z
tags: monitoring, logging, security
editor: markdown
dateCreated: 2026-09-09T10:00:00.000Z
---

# 02. Log Data Dictionary

## 개념

Log Data Dictionary는 `어떤 상황에서 어떤 정보를 반드시 로그로 남길지` 정리한 기준표.

보안팀이 정의한 항목은 주로 침해사고 대응, 감사, 이상 행위 탐지에 필요한 필수 이벤트와 필드.

예시

| 이벤트 | 필요한 정보 |
|---|---|
| 로그인 | 사용자 ID, 성공/실패, IP, User-Agent, 시각 |
| 권한 변경 | 대상 리소스, 변경 전/후 권한, 관리자 액션 |
| 수요 등록 | 사용자 ID, 상품 카테고리, 목표 수량, 마감일, IP |
| 셀러 조건 변경 | 셀러 ID, 수요 ID, 변경 전/후 값, 변경 시각 |
| 공동구매 마감 | 수요 ID, 마감 유형, 참여 인원, 마감 시각 |

## Dictionary와 실제 JSON 필드의 관계

Dictionary의 한글 항목을 그대로 JSON Key로 사용할 필요는 없음.

개발팀과 공통 Naming Rule을 정해서 실제 필드명으로 변환.

| Dictionary 항목 | JSON Key 예시 |
|---|---|
| 사용자ID | `user_id` |
| 성공/실패 | `result` |
| IP | `client_ip` |
| User-Agent | `user_agent` |
| 시각 | `timestamp` |
| 응답 코드 | `status_code` |

## 공통 필드 + 이벤트별 필드

모든 로그에 필요한 공통 필드와 특정 이벤트에서만 필요한 필드를 분리.

```text
공통 필드
├─ timestamp
├─ level
├─ service_name
├─ environment
├─ event
├─ trace_id
└─ span_id

이벤트별 필드
├─ login → user_id, result, client_ip
├─ demand_created → product_category, target_quantity
└─ seller_condition_changed → before, after, seller_id
```

## 민감정보 처리

`개인정보 접근 로그`는 개인정보 원문을 남긴다는 뜻이 아님.

나쁜 예

```json
{
  "card_number": "1234-5678-9012-3456"
}
```

권장 형태

```json
{
  "event": "personal_data_access",
  "user_id": "user-128",
  "resource_type": "payment_info",
  "action": "READ",
  "result": "SUCCESS"
}
```

## 핵심

```text
보안팀 Data Dictionary
        ↓
무슨 이벤트와 정보를 남길지 결정
        ↓
개발/DevOps 공통 JSON Schema로 변환
```

다음: [03. Structured JSON Log](/Monitoring/Logging/03-JSON-Log-Format)
