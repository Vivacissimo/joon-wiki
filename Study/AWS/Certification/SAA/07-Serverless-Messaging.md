---
title: SAA 07 — 서버리스와 메시징
description: SQS, SNS, EventBridge, 스트림과 Lambda의 재시도·멱등성 설계
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 서버리스와 메시징

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 메시지를 왜 넣는가?

```text
동기식: 사용자 → 주문 API → 재고 → 이메일 → 응답

비동기: 사용자 → 주문 API → 큐 → Worker → 이메일
                    ↓
                 빠른 접수 응답
```

큐는 생산자와 소비자의 처리 속도를 분리하고 일시적인 트래픽을 완충한다. 큐가 생긴다고 처리 용량이 무한해지지는 않는다. 적체 시간, 소비자 확장, 실패 재처리, 사용자에게 보여 줄 처리 상태가 필요하다.

## 2. 서비스 비교

| 서비스 | 중심 목적 | 선택 단서 |
|---|---|---|
| SQS | 작업을 보관하고 소비자가 처리 | 버퍼링, 작업 분산 |
| SNS | 메시지를 여러 구독 대상으로 전달 | Fan-out, 여러 독립 소비자 |
| EventBridge | 이벤트 규칙에 따라 대상 라우팅 | AWS·SaaS·사용자 이벤트 연결 |
| Step Functions | 상태를 가진 다단계 워크플로 | 분기·재시도·단계별 추적 |
| Amazon MQ | 관리형 메시지 브로커 | 기존 브로커·프로토콜 호환성 유지 |

한 SQS 큐에 여러 Worker가 붙으면 경쟁 소비한다. 각 하위 시스템이 이벤트를 독립적으로 모두 받아야 한다면 SNS → 시스템별 SQS처럼 구독별 큐를 둔다. [통합 서비스 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/application-integration.html)

## 3. SQS Standard와 FIFO

| 비교 | Standard | FIFO |
|---|---|---|
| 순서 | 최선 노력, 순서 달라질 수 있음 | 같은 Message Group 내 순서 보장 |
| 중복 | 최소 1회 전달, 중복 가능 | 송신 중복 제거 기능 |
| 사용 예 | 독립적인 이미지 처리 | 주문별 순서가 중요한 이벤트 |

Standard는 소비자의 멱등성이 중요하다. FIFO도 메시지 수신 후 삭제 전에 처리기가 실패하면 재전달될 수 있다. **메시지 중복 제거가 외부 결제의 정확히 한 번 실행을 자동 보장하지 않는다.** 주문 ID·이벤트 ID와 조건부 기록 등으로 중복 효과를 막는다. [Standard 전달 특성](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html), [FIFO 중복 제거](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues-exactly-once-processing.html)

## 4. SQS 설정에서 구분할 것

| 설정 | 의미 | 흔한 혼동 |
|---|---|---|
| Visibility Timeout | 받은 메시지가 다른 소비자에게 잠시 보이지 않는 시간 | 메시지 보관 기간과 다름 |
| Retention Period | 메시지 보관 기간 | 작업 실행 시간 제한이 아님 |
| Long Polling | 메시지를 기다려 빈 응답·불필요한 호출 감소 | 메시지 전달을 일부러 늦추는 배치 설정이 아님 |
| DLQ | 반복 실패한 메시지 분리 | 원인을 수정하고 재처리해야 함 |
| DeleteMessage | 성공적으로 처리한 메시지 삭제 | 수신만으로 삭제되지 않음 |

작업이 Visibility Timeout보다 오래 걸리면 처리 중인 메시지가 다시 보일 수 있다. 타임아웃과 연장 전략을 맞추고, 무조건 긴 값은 장애 재처리를 늦출 수 있음을 기억한다. [SQS Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)

## 5. 스트림과 전달 서비스

| 요구 | 후보 | 판단 기준 |
|---|---|---|
| 연속 이벤트를 여러 앱이 읽고 재처리 | Kinesis Data Streams | 보존·순서·처리량·파티션 키 |
| 데이터를 S3 등 대상으로 관리형 전달 | Amazon Data Firehose | 버퍼링·변환·전달 지연 |
| 기존 Kafka 생태계 활용 | Amazon MSK | Kafka 호환과 클러스터 운영 방식 |
| 스트림의 상태 기반 실시간 분석 | Managed Service for Apache Flink | 윈도·집계·상태 처리 |

스트림의 순서는 보통 샤드·파티션 같은 범위와 연결된다. 모든 이벤트의 전역 순서를 기본 보장한다고 외우지 않는다. Firehose는 사용자 정의 소비자가 자유롭게 재생하는 스트림 저장소와 역할이 다르다. [분석 서비스 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/analytics.html)

## 6. 서버리스 API 구성

```text
클라이언트 → API Gateway → Lambda → DynamoDB
                             ↓
                            SQS → 후속 처리
```

API Gateway는 API 진입점, Lambda는 로직 실행, DynamoDB는 데이터 저장이다. 인증·스로틀링·재시도·동시성·하위 DB 용량을 따로 설계한다. Lambda 동시성 제한은 비용 제어뿐 아니라 DB 연결 등 하위 자원 보호에도 관련된다. [Lambda 동시성](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)

이벤트 소스에 따라 재시도와 실패 처리 방식이 다르다. “Lambda가 실패하면 항상 같은 횟수로 재시도”라고 일반화하지 않는다. [Lambda 오류 처리](https://docs.aws.amazon.com/lambda/latest/dg/invocation-retries.html)

다음: [분석·이전·재해 복구](/en/Study/AWS/Certification/SAA/08-Analytics-Migration-DR)
