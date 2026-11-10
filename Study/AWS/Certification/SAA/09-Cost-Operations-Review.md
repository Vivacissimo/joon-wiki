---
title: SAA 09 — 비용·운영과 최종 비교표
description: 구매 옵션, 관측·감사 서비스와 요구사항별 최종 선택 기준
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 비용·운영과 최종 비교표

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 비용 최적화의 순서

필수 보안·성능·가용성 조건을 먼저 만족시킨 뒤 비용을 비교한다. 요구사항을 만족하지 못하는 가장 싼 구성은 정답 후보가 아니다.

1. 사용하지 않는 리소스와 과도한 사양을 줄인다.
2. 수요에 맞춰 용량을 조절하고 유휴 시간을 줄인다.
3. 사용 패턴이 안정되면 약정 할인을 검토한다.
4. 저장·요청·전송·복원·라이선스·운영 비용을 함께 비교한다.

## 2. EC2 구매 옵션

| 옵션 | 적합한 조건 | 주의 |
|---|---|---|
| On-Demand | 단기·불확실한 사용량 | 장기 약정 할인 없음 |
| Savings Plans | 일정 금액의 시간당 사용을 1년·3년 약정 | 약정한 사용료와 적용 범위 확인 |
| Reserved Instances | 일정 인스턴스 구성의 지속 사용 | 옵션별 유연성과 용량 예약 차이 |
| Spot | 중단·재시도 가능한 작업 | 중단 대응·체크포인트 필요 |
| Dedicated Hosts | 물리 호스트 단위 제어·라이선스 | 단순한 격리 요구와 구분 |
| Capacity Reservations | 특정 AZ의 실행 용량 확보 | 할인 약정과 용량 보장은 별개 |

Savings Plans만으로 실행 용량이 예약되지는 않는다. Reserved Instances도 리전·AZ 범위에 따라 용량 예약 여부가 다르다. 실제 사용 패턴과 배포 AZ를 확인한다. [EC2 구매 옵션](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)

## 3. 숨은 비용을 찾는 기준

| 항목 | 확인할 것 |
|---|---|
| 네트워크 | 인터넷 송신, 교차 AZ·리전 전송, NAT 처리, 엔드포인트 |
| S3 | 객체 용량, 요청, 검색·복원, 최소 보관 기간 |
| DB | 인스턴스·용량, I/O, 복제본·백업 |
| 서버리스 | 호출량·실행 시간·메모리, 로그·하위 서비스 |
| 운영 | 패치·백업·복구·장애 대응의 인력 부담 |

S3만 접근하는 프라이빗 워크로드는 Gateway Endpoint로 NAT 경유를 줄일 수 있다. 그러나 모든 Interface Endpoint가 무료인 것은 아니다. [S3 Gateway Endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)

가격 수치보다 과금 단위를 먼저 이해한다. 리전·사용량에 따라 달라지는 금액은 [AWS Pricing Calculator](https://calculator.aws/)에서 실제 조건으로 산정한다.

## 4. 비용 관리 도구

| 도구 | 목적 |
|---|---|
| Cost Explorer | 사용량·비용 추세와 원인 분석 |
| AWS Budgets | 예산·사용량 기준 알림과 구성된 작업 |
| Cost Allocation Tags | 프로젝트·팀 단위 비용 분류 |
| Cost Anomaly Detection | 비정상적 비용 패턴 탐지 |

예산 알림만 설정하면 모든 리소스가 자동 중지되는 것은 아니다. 알림과 자동 조치는 별도 구성을 확인한다. [AWS 비용 관리 개요](https://docs.aws.amazon.com/cost-management/latest/userguide/what-is-costmanagement.html)

## 5. 모니터링·감사·구성 관리

| 서비스 | 중심 질문 | 용도 |
|---|---|---|
| CloudWatch | 지금 시스템 상태는 어떤가? | 지표·로그·알람 |
| CloudTrail | 누가 어떤 API 작업을 했는가? | 활동 기록·감사 |
| AWS Config | 리소스 구성이 어떻게 바뀌었는가? | 구성 이력·규칙 준수 평가 |
| Systems Manager | 운영 작업을 어떻게 관리할 것인가? | 세션·명령·패치 등 |
| CloudFormation | 인프라를 어떻게 반복 배포할 것인가? | 코드로 자원 정의·관리 |

CloudTrail은 기록하도록 설정한 이벤트 범위를 확인한다. Config의 비준수 탐지와 실제 자동 수정도 구분한다. [관리·거버넌스 서비스](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/management-governance.html)

## 6. 헷갈리는 선택 최종 비교

| 비교 | 판단 기준 |
|---|---|
| ALB / NLB | HTTP 호스트·경로 라우팅 / 전송 계층·고정 IP 요구 |
| CloudFront / Global Accelerator | 웹 배포·캐시 / 글로벌 TCP·UDP 경로 |
| S3 / EFS / EBS | 객체 API / 공유 파일 / AZ 단위 블록 |
| Multi-AZ DB instance / Read Replica | 장애 전환 / 읽기 확장 |
| RDS Proxy / ElastiCache | 연결 재사용 / 데이터 조회 캐시 |
| SQS / SNS | 작업 대기열 / 여러 구독 대상 전달 |
| EventBridge / Step Functions | 이벤트 라우팅 / 단계·상태 관리 |
| Athena / Redshift | S3의 간헐적 SQL / 웨어하우스 분석 |
| DataSync / Storage Gateway | 데이터 전송 / 하이브리드 저장 접근 |
| WAF / GuardDuty | 웹 요청 제어 / 위협 탐지 |
| CloudWatch / CloudTrail / Config | 상태 / API 활동 / 구성 |

표는 앞 문서의 요약이다. 예외와 세부 조건은 각 주제 문서의 공식 출처를 확인한다.

다음: [최종 복습 체크리스트](/en/Study/AWS/Certification/SAA/10-Readiness)
