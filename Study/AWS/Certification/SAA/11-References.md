---
title: SAA 11 — 참고 자료와 검증 기준
description: 사용자 제공 블로그의 활용 범위와 AWS 공식 문서 확인 기준
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 참고 자료와 검증 기준

[SAA 목차](/en/Study/AWS/Certification/SAA)

확인 기준일: **2026-11-10**.

## 1. 제공된 블로그의 활용 범위

| 자료 | 확인한 범위 | 이 문서에 활용한 부분 |
|---|---|---|
| [찰리 킴 — SAA-C03 준비 과정 및 시험 후기](https://charliekim.tistory.com/5) | 제목·게시일은 확인했으나 본문 조회 불가 | 참고 링크로 보존. 학습 기간·교재·후기 내용은 인용하지 않음 |
| [N(e)xtCloud](https://pacloud.tistory.com/) | 홈의 SAA 글 목록과 공개 미리보기. 개별 최신 글 본문은 조회 불가 | 요구사항 분석 → 관련 서비스 검토 → 선택지 비교의 학습 접근 |
| [G.log — AWS SAA TIL](https://velog.io/@gagaeun/series/AWS-SAA-TIL) | 시리즈 목차·게시 시점과 VPC 글 | 서비스별 주제 구성과 네트워크 학습 범위 참고 |

블로그 전체를 읽거나 그대로 옮긴 요약본은 아니다. 학습 구성을 참고하고 기술 설명은 공식 문서를 기준으로 독립적으로 작성했다. 참고 시리즈에는 SAA와 DVA 내용이 함께 있으므로 시리즈에 등장하는 모든 항목을 현재 SAA 필수 범위로 간주하지 않는다.

## 2. 시험 범위의 기준

- [SAA-C03 공식 시험 가이드](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html)
- [공식 출제 범위 내 서비스](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html)
- [공식 시험·준비 안내](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

핵심 노트에서 상세히 설명하지 않은 서비스도 출제 범위에 있을 수 있다. 공식 목록과 대조하여 부족한 주제를 보충한다. 이 문서의 순서나 분량을 실제 출제 빈도 통계로 해석하지 않는다.

## 3. 기술 설명의 검증 기준

각 주제 문서의 문장·표 가까이에 AWS 공식 문서 링크를 배치했다. 충돌하는 설명이 있으면 현재 공식 문서에서 서비스 구성·리전·버전 조건을 확인한다.

| 오래된 요약에서 주의할 표현 | 확인할 내용 |
|---|---|
| Multi-AZ는 읽기 불가 | DB instance와 DB cluster 배포 유형 구분 |
| Global Tables는 항상 최종 일관성 | 현재 지원하는 일관성 모드와 조건 |
| Lambda는 예외 없이 15분 | 일반 호출과 특정 실행 모델의 제약 구분 |
| 모든 S3 Glacier는 즉시 조회 불가 | Instant Retrieval과 복원이 필요한 계층 구분 |
| Fargate는 ECS와 경쟁하는 오케스트레이터 | 오케스트레이션과 실행 용량 구분 |
| FIFO면 결제도 정확히 한 번 실행 | 메시지 중복 제거와 외부 효과의 멱등성 구분 |
| Snow 장비는 누구나 신규 주문 가능 | 현재 고객 자격·서비스 가용성 확인 |

## 4. 문서를 보완하는 방법

새 내용을 추가할 때는 서비스 이름만 늘리지 않고 다음 네 가지를 기록한다.

1. 해결하는 요구사항.
2. 비슷한 서비스와의 차이.
3. 선택하면 안 되는 조건과 주요 제약.
4. 확인한 공식 출처와 확인 날짜.

가격·할당량·지원 리전은 바뀔 수 있다. 모든 수치를 외우기보다 문제에 필요한 제약을 파악하고 응시 전에 현재 안내를 확인한다.
