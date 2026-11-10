---
title: SAA 06 — 데이터베이스와 캐시
description: 관계형 DB, DynamoDB, 읽기 확장, 장애 복구와 캐시 비교
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 데이터베이스와 캐시

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 데이터 모델과 접근 패턴

| 요구사항 | 후보 | 이유 |
|---|---|---|
| 관계·조인·SQL·트랜잭션 | RDS / Aurora | 관계형 모델 |
| 키 중심의 대규모 조회·쓰기 | DynamoDB | 키-값·문서 모델 |
| 반복 조회를 메모리에서 처리 | ElastiCache | 원본 DB 부하와 지연 감소 |
| 대량 분석 집계 | Redshift | 분석용 데이터 웨어하우스 |
| 관계 탐색·그래프 순회 | Neptune | 그래프 모델 |
| MongoDB 호환성 요구 | DocumentDB 검토 | 호환 범위·기능 차이 확인 |

“NoSQL이 항상 빠르다”보다 실제 쿼리·일관성·트랜잭션·운영 조건을 먼저 본다. 서비스가 같은 프로토콜을 지원해도 모든 기능이 완전히 호환되는 것은 아니다. [AWS 데이터베이스 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/database.html)

## 2. RDS에서 가장 중요한 구분

| 구성 | 주 목적 | 읽기 처리 | 복제·장애 전환 |
|---|---|---|---|
| Multi-AZ DB instance | 고가용성 | 대기 인스턴스는 읽기 제공 안 함 | 동기식 대기 복제·자동 장애 전환 |
| Multi-AZ DB cluster | 고가용성과 읽기 처리 | 읽기 가능한 대기 인스턴스 | 구성·지원 엔진 조건 확인 |
| Read Replica | 읽기 확장·보고 쿼리 분리 | 복제본에서 읽기 | 일반 RDS 인스턴스 복제는 비동기, 지연 가능 |

**“Multi-AZ는 읽기 확장 불가”는 DB instance 구성에 대한 설명이다.** DB cluster 구성까지 같은 말로 묶지 않는다. [Multi-AZ 공식 구분](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

읽기 복제본은 쓰기를 분산하는 기능이 아니다. 복제본을 만들기만 해서는 애플리케이션의 읽기 쿼리가 자동으로 이동하지 않으므로 연결 대상을 조정해야 한다. 일반 Read Replica의 승격과 Multi-AZ 자동 장애 전환도 구분한다. [RDS Read Replica](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)

## 3. Aurora

Aurora는 컴퓨팅과 분산 스토리지를 분리한다. 저장 계층이 다중 AZ에 복제되어 있다는 것과 장애 시 즉시 서비스를 이어받을 DB 인스턴스가 준비되어 있다는 것은 다른 조건이다. 여러 AZ의 Aurora Replica를 통해 인스턴스 장애 복구를 설계한다. Reader Endpoint는 읽기 연결 분산을 돕고, 쓰기는 Writer Endpoint로 보낸다. [Aurora 고가용성](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)

| 기능 | 선택 단서 |
|---|---|
| Aurora Replica | 읽기 처리·장애 전환 후보 |
| Aurora Global Database | 리전 간 복제·글로벌 읽기·DR |
| Aurora Serverless | 가변적인 DB 수요와 용량 관리 부담 |
| RDS Proxy | 연결 풀링·연결 급증 완화 |

RDS Proxy는 쿼리 캐시나 읽기 복제본이 아니다. Lambda가 동시에 많은 연결을 여는 문제와 DB가 느린 쿼리를 처리하는 문제를 구분한다. [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)

## 4. DynamoDB

| 항목 | 기억할 것 |
|---|---|
| Partition Key | 데이터와 요청 분산에 중요, 편향되면 핫 파티션 위험 |
| Sort Key | 같은 파티션 키 내부 정렬·범위 조회 설계 |
| Query | 키 조건 중심 조회 |
| Scan | 전체를 읽는 방식, 큰 테이블에서 비용·성능 주의 |
| On-demand | 용량 계획 부담을 줄이는 요청 기반 방식 |
| Provisioned | 예측 가능한 처리량에 용량 설정·확장 구성 |
| TTL | 만료 항목 자동 정리, 정각 즉시 삭제 보장으로 가정 금지 |
| Streams | 항목 변경을 이벤트 처리로 연결 |
| DAX | DynamoDB 읽기 캐시 |

기본 읽기는 최종 일관성이다. 테이블·LSI는 강한 일관성 읽기를 선택할 수 있지만 GSI는 지원하지 않는다. 방금 쓴 값을 반드시 읽어야 하는 요구와 캐시의 오래된 값 가능성을 함께 고려한다. [DynamoDB 읽기 일관성](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)

Global Tables는 여러 리전에서 읽기·쓰기를 제공한다. 현재는 최종 일관성과 다중 리전 강한 일관성 모드가 있으므로 “Global Tables는 언제나 최종 일관성”이라는 과거 설명을 그대로 적용하지 않는다. 모드별 리전·기능 제약을 확인한다. [Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)

## 5. 캐시 전략

| 전략 | 동작 | 주의 |
|---|---|---|
| Cache-aside / Lazy Loading | 캐시 미스 → DB 조회 → 캐시 채움 | 첫 요청 지연, 만료·무효화 필요 |
| Write-through | 쓰기 시 캐시도 갱신 | 읽지 않는 데이터도 캐싱, 실패 처리 고려 |
| TTL | 일정 시간 뒤 만료 | 데이터가 오래될 수 있는 시간 허용 필요 |

캐시를 붙이면 “항상 최신” 조건을 자동으로 만족하는 것은 아니다. 원본 저장소·무효화·장애 시 처리 방식을 정한다. ElastiCache의 엔진별 복제·고가용성·자료구조 지원 차이도 확인한다. [캐시 전략](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Strategies.html)

다음: [서버리스와 메시징](/en/Study/AWS/Certification/SAA/07-Serverless-Messaging)
