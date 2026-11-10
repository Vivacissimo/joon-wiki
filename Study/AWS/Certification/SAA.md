---
title: SAA — Solutions Architect Associate
description: SAA-C03 시험 준비를 위한 핵심 개념, 서비스 비교표와 복습 계획
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# AWS SAA 학습 가이드

SAA는 요구사항에 맞는 AWS 아키텍처를 선택하는 시험이다. 서비스를 설명할 수 있는 것에서 더 나아가, **이 조건에서 왜 이 서비스를 선택하고 다른 서비스는 제외하는지** 말할 수 있어야 한다.

확인 기준일: **2026-11-10**. 공식 시험 가이드 기준 시험 코드는 **SAA-C03**이다. 응시 전에는 아래 공식 페이지에서 시험 버전과 안내 변경 여부를 다시 확인한다.

## 1. 시험 정보

| 항목 | 내용 |
|---|---|
| 문항 / 시간 | 65문항 / 130분 |
| 유형 | 단일 선택, 복수 선택 |
| 채점 | 채점 대상 50문항 + 구분되지 않는 비채점 15문항 |
| 합격 | 100~1,000점 척도에서 720점 이상 |
| 미응답 | 오답 처리, 추측에 대한 별도 감점 없음 |
| 언어 | 한국어 지원 |

720점은 단순 정답률 72%라는 뜻이 아니다. 합격 여부는 전체 환산 점수로 판정한다. [공식 시험 가이드](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html), [시험 안내](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

## 2. 시험 영역과 연결 문서

| 영역 | 비중 | 주로 연결되는 문서 |
|---|---:|---|
| 보안 아키텍처 | 30% | IAM·보안, 네트워크, 스토리지 |
| 복원력 있는 아키텍처 | 26% | 컴퓨팅, DB, 비동기 처리, 재해 복구 |
| 고성능 아키텍처 | 24% | 컴퓨팅, 스토리지, DB, 분석 |
| 비용 최적화 아키텍처 | 20% | 비용·운영, 각 서비스 비교표 |

비중은 공식 가이드 기준이다. 아래 문서의 학습 순서는 이해를 돕기 위한 편집 순서이며 실제 출제 빈도 순위가 아니다. [영역별 비중](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html#content-outline)

## 3. 학습 목차

| 순서 | 문서 | 학습 후 할 수 있어야 하는 것 |
|---|---|---|
| 01 | [학습 계획과 문제 풀이 방법](/en/Study/AWS/Certification/SAA/01-Study-Plan) | 조건 추출, 오답 분류, 반복 복습 |
| 02 | [IAM과 보안](/en/Study/AWS/Certification/SAA/02-IAM-Security) | 역할·정책·암호화·보안 서비스 선택 |
| 03 | [VPC와 글로벌 네트워크](/en/Study/AWS/Certification/SAA/03-Networking) | 통신 경로와 접근 제어 설명 |
| 04 | [컴퓨팅과 고가용성](/en/Study/AWS/Certification/SAA/04-Compute-HA) | EC2·컨테이너·Lambda, ALB·ASG 선택 |
| 05 | [스토리지](/en/Study/AWS/Certification/SAA/05-Storage) | 블록·파일·객체와 보관 계층 선택 |
| 06 | [데이터베이스와 캐시](/en/Study/AWS/Certification/SAA/06-Databases) | 장애 복구·읽기 확장·일관성 구분 |
| 07 | [서버리스와 메시징](/en/Study/AWS/Certification/SAA/07-Serverless-Messaging) | 큐·발행 구독·이벤트·스트림 구분 |
| 08 | [분석·이전·재해 복구](/en/Study/AWS/Certification/SAA/08-Analytics-Migration-DR) | 데이터 처리와 RTO·RPO에 맞는 설계 |
| 09 | [비용·운영과 최종 비교표](/en/Study/AWS/Certification/SAA/09-Cost-Operations-Review) | 조건을 만족하는 설계 중 비용·운영 비교 |
| 10 | [최종 복습 체크리스트](/en/Study/AWS/Certification/SAA/10-Readiness) | 핵심 개념의 이해 여부 점검 |
| 11 | [참고 자료와 검증 기준](/en/Study/AWS/Certification/SAA/11-References) | 공식 범위 확인, 오래된 설명 점검 |

## 4. 공부하는 방법

1. 02~08의 개념을 읽고 비교표를 가린 채 차이를 설명한다.
2. 시나리오 문제를 풀 때 정답 근거와 오답 제외 이유를 함께 적는다.
3. 틀리거나 추측한 문제는 오답 노트에 기록하고 1·3·7일 뒤 다시 푼다.
4. 처음 보는 모의문제를 시간 제한 안에 풀어 준비도를 확인한다.

이 문서는 핵심 학습 노트이며 모든 출제 서비스를 상세히 다룬 완전한 교재는 아니다. 마지막에는 [공식 출제 범위 목록](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html)과 대조한다. 합격은 암기량만으로 보장되지 않으므로 실제 문제 풀이와 취약 영역 보완까지 진행한다.
