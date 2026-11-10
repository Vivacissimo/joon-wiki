---
title: SAA 10 — 최종 복습 체크리스트
description: 영역별 이해 점검과 시험 전 복습 순서
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 최종 복습 체크리스트

[SAA 목차](/en/Study/AWS/Certification/SAA)

체크 표시는 읽었다는 뜻이 아니라, 자료 없이 선택 이유와 한계를 설명할 수 있다는 뜻으로 사용한다.

## 1. 보안

- [ ] 역할의 신뢰 정책과 작업 권한을 구분한다.
- [ ] 명시적 Deny, SCP, Permissions Boundary의 역할을 설명한다.
- [ ] S3 접근 권한과 KMS 복호화 권한을 함께 확인한다.
- [ ] 직원용 Identity Center와 고객 앱용 Cognito를 구분한다.
- [ ] WAF·Shield·GuardDuty·Inspector·Macie의 목적을 구분한다.
- [ ] 저장 중 암호화와 전송 중 암호화를 구분한다.

## 2. 네트워크

- [ ] 퍼블릭·프라이빗 서브넷의 라우트를 그릴 수 있다.
- [ ] SG·NACL의 상태 추적과 규칙 평가 차이를 설명한다.
- [ ] NAT와 VPC Endpoint의 통신 목적·비용 차이를 설명한다.
- [ ] Peering의 비전이성과 Transit Gateway의 역할을 설명한다.
- [ ] VPN과 Direct Connect에서 전용 연결·암호화를 구분한다.
- [ ] CloudFront·Global Accelerator·Route 53의 역할을 구분한다.

## 3. 복원력과 컴퓨팅

- [ ] 로드 밸런서 대상 제외와 ASG 인스턴스 교체를 구분한다.
- [ ] 예약 확장·동적 확장과 워밍업을 설명한다.
- [ ] 다중 AZ와 다중 리전의 장애 범위를 구분한다.
- [ ] 로컬 세션·파일이 수평 확장에 주는 영향을 설명한다.
- [ ] ECS·EKS·Fargate·Lambda의 선택 기준을 설명한다.

## 4. 스토리지와 데이터

- [ ] 객체·파일·블록 접근 방식으로 저장소를 고른다.
- [ ] EBS의 AZ 범위와 EFS 배포 유형을 구분한다.
- [ ] S3 보관 계층을 조회 지연·검색 비용·보관 기간으로 비교한다.
- [ ] 버전 관리·복제·백업·Object Lock의 목적을 구분한다.
- [ ] RDS Multi-AZ의 두 배포 유형과 Read Replica를 구분한다.
- [ ] DynamoDB 키 설계·일관성·인덱스 제약을 설명한다.
- [ ] 캐시의 무효화·TTL·오래된 값 가능성을 설명한다.

## 5. 통합·복구·운영

- [ ] 큐·발행 구독·이벤트·스트림을 구분한다.
- [ ] Visibility Timeout, DLQ, 멱등성이 필요한 이유를 설명한다.
- [ ] 분석·DB 이전·파일 이전 서비스의 목적을 구분한다.
- [ ] RTO와 RPO를 분리하고 DR 전략을 비교한다.
- [ ] 할인 약정과 용량 예약을 구분한다.
- [ ] CloudWatch·CloudTrail·Config와 비용 관리 도구를 구분한다.

## 6. 마지막 복습 순서

1. 체크하지 못한 항목을 해당 문서에서 다시 읽는다.
2. 비교표를 가리고 각 서비스의 선택 조건과 제외 조건을 설명한다.
3. [공식 출제 범위](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html)에서 아직 모르는 서비스 이름을 표시한다.
4. 빠진 서비스는 공식 개요에서 목적·대체재·선택 조건을 보충한다.
5. 공식 연습 자료로 시간 안에 요구사항을 해석할 수 있는지 확인한다.

이 체크리스트 완료만으로 합격을 보장하지 않는다. 실제 준비도는 처음 보는 시나리오에서 조건을 읽고 선택 이유를 설명할 수 있는지로 판단한다. [AWS 공식 준비 안내](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

다음: [참고 자료와 검증 기준](/en/Study/AWS/Certification/SAA/11-References)
