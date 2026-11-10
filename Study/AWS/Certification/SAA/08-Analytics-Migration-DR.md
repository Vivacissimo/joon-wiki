---
title: SAA 08 — 분석·이전·재해 복구
description: 분석 서비스 선택, 데이터·서버 이전과 RTO·RPO 기반 복구 전략
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 분석·이전·재해 복구

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 분석 서비스 선택

| 요구 | 서비스 | 비교 포인트 |
|---|---|---|
| S3 데이터를 SQL로 간헐적 분석 | Athena | 스캔량, 파티션·열 형식 |
| ETL·데이터 카탈로그 | Glue | 데이터 변환·스키마 관리 |
| 데이터 레이크 권한 관리 | Lake Formation | 분석 데이터 접근 거버넌스 |
| 대규모 집계·BI용 웨어하우스 | Redshift | 운영 형태와 쿼리 패턴 |
| Spark 등 분산 처리 | EMR | 처리 프레임워크 요구 |
| 검색·로그 탐색 | OpenSearch Service | 검색 인덱스와 집계 |

서비스 선택 후에는 스캔 범위를 줄이는 파티셔닝과 Parquet 같은 열 기반 형식도 검토한다. “서버리스이니 항상 가장 싸다”보다 실제 읽는 데이터량과 실행 패턴이 중요하다. [분석 서비스 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/analytics.html)

## 2. 이전 서비스 선택

| 무엇을 옮기는가? | 후보 | 조건 |
|---|---|---|
| DB 데이터와 지속 변경 | DMS | 소스·대상 지원과 CDC 구성 |
| 다른 DB 엔진으로 스키마 변환 | DMS Schema Conversion 등 | 자동 변환 안 되는 객체 검토 |
| 파일·객체 온라인 전송 | DataSync | 지원 위치·전송 경로·에이전트 |
| SFTP 등 기존 파일 전송 인터페이스 | Transfer Family | 클라이언트 프로토콜 유지 |
| 기존 서버를 큰 수정 없이 이전 | AWS MGN 계열 서비스 | 서버 복제·전환 계획 |
| 오프라인 대용량 데이터 반입 | Snow 계열 검토 | 신규 주문 자격·가용성·현재 대안 확인 |

DMS는 원본 DB를 계속 운영하면서 변경 데이터를 반영하는 이전을 지원한다. 다운타임을 줄일 수 있지만 모든 엔진·객체가 변경 없이 자동 이전되는 것은 아니다. [이전 서비스 공식 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/migration-services.html)

Snow 관련 과거 학습 자료는 현재 주문 가능 여부와 다를 수 있다. “느린 회선과 큰 데이터”라는 개념적 단서와 실제 사용할 수 있는 서비스를 구분한다. [Snow Family 안내](https://aws.amazon.com/snow/)

## 3. Storage Gateway

온프레미스 애플리케이션이 기존 파일·볼륨·테이프 인터페이스를 유지하면서 AWS 저장소와 연결해야 할 때 검토한다. DataSync는 데이터를 옮기는 작업, Storage Gateway는 하이브리드 저장 접근이라는 관점으로 구분한다. [Storage Gateway](https://docs.aws.amazon.com/storagegateway/latest/userguide/WhatIsStorageGateway.html)

## 4. RTO와 RPO

| 용어 | 질문 | 예 |
|---|---|---|
| RTO | 장애 후 얼마나 빨리 복구해야 하는가? | 30분 안에 서비스 재개 |
| RPO | 어느 시점까지의 데이터 손실을 허용하는가? | 최대 5분치 손실 |

백업을 1시간마다 한다고 복구도 1시간 안에 끝나는 것은 아니다. 백업 주기는 데이터 손실 범위와 관련되고, 실제 복구 시간은 인프라 재구성·복원·DNS 전환·검증에 달려 있다.

## 5. DR 전략 비교

| 전략 | 평소 준비 상태 | 비용·복구 방향 |
|---|---|---|
| Backup & Restore | 백업을 보관, 장애 후 환경 복구 | 낮은 상시 비용, 상대적으로 긴 복구 |
| Pilot Light | 데이터 등 핵심 요소 유지 | 장애 후 나머지 계층 생성·확장 |
| Warm Standby | 축소된 완전한 환경 실행 | 용량 확장 후 전환 |
| Multi-site Active/Active | 여러 사이트에서 실제 트래픽 처리 | 높은 비용·복잡도, 짧은 복구 목표에 대응 |

이는 상대적 방향이며 전략 이름만으로 특정 RTO·RPO를 보장하지 않는다. 장애 범위와 실제 복구 테스트 결과가 중요하다. [AWS DR 전략](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)

## 6. 백업과 복제의 차이

복제는 가용성과 데이터 전달을 돕는다. 논리적 오류·잘못된 쓰기가 함께 복제될 수 있으므로 이전 시점 복구를 위한 백업도 필요하다. 백업은 데이터 외에 인프라 코드·설정·키 접근 권한까지 함께 복원할 수 있어야 한다.

다중 리전 DR에서는 복제 지연, 키·비밀의 가용성, 장애 리전 없이 전환 가능한지, 리전별 서비스 지원을 확인한다. 주기적으로 복구 순서를 실행하고 측정한다. [DR 복구 고려사항](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)

다음: [비용·운영과 최종 비교표](/en/Study/AWS/Certification/SAA/09-Cost-Operations-Review)
