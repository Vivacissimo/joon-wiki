---
title: SAA 02 — IAM과 보안
description: 역할과 정책 평가, 암호화, 다중 계정과 보안 서비스 선택 기준
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# IAM과 보안

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 인증과 권한을 나누기

인증은 “누구인가”, 권한 부여는 “무엇을 할 수 있는가”이다. 역할을 맡을 수 있는 것과 그 역할로 S3를 읽을 수 있는 것은 별도의 판단이다.

| 개념 | 역할 | 선택 단서 |
|---|---|---|
| IAM Role | 임시 자격 증명으로 작업하는 주체 | EC2·Lambda·다른 계정의 접근 |
| Trust Policy | 누가 역할을 맡을 수 있는지 정의 | AssumeRole 허용 대상 |
| Permissions Policy | 어떤 작업을 어떤 리소스에 할 수 있는지 정의 | S3 읽기, KMS 복호화 |
| STS | 임시 보안 자격 증명 발급 | 교차 계정·연합 인증 |
| IAM Identity Center | 직원의 여러 AWS 계정·앱 접근 관리 | 사내 사용자 SSO |
| Cognito | 고객용 웹·모바일 앱 인증 등 | 회원가입·로그인 |

EC2에서 S3를 읽는다면 인스턴스 역할을 사용한다. 코드나 AMI에 장기 액세스 키를 넣는 방식은 자격 증명 교체와 유출 대응 부담이 크다.

## 2. 정책 평가의 핵심

- 명시적 Deny는 Allow보다 우선한다.
- SCP와 Permissions Boundary는 권한의 상한을 제한하며 그 자체로 작업 권한을 부여하지 않는다.
- 같은 계정의 자격 증명 정책과 리소스 정책은 함께 평가된다. 단순히 “둘 다 Allow가 있어야 한다”로 외우지 않는다.
- 교차 계정은 호출자 측 권한과 대상 계정의 신뢰·리소스 허용을 함께 확인한다.

리소스 정책이 사용자 ARN, 역할 ARN, 역할 세션 중 누구를 대상으로 하는지에 따라 세부 평가는 달라질 수 있다. 접근 거부 문제에서는 **명시적 Deny → 조직 정책·경계 → 역할·리소스 정책 → 서비스별 조건** 순서로 점검한다. [정책 평가 공식 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)

### 예시: 암호화된 S3 객체가 안 읽힌다

`s3:GetObject`만 확인하면 부족하다. SSE-KMS 객체라면 KMS 복호화 권한과 키 정책도 확인한다. 네트워크가 연결되어 있어도 IAM·버킷·키 정책 중 하나가 접근을 막을 수 있다.

## 3. 암호화와 비밀 관리

| 요구사항 | 후보 | 확인할 조건 |
|---|---|---|
| 저장 데이터 암호화와 키 제어 | KMS | 키 정책, 사용 권한, 감사 |
| HTTPS 인증서 관리 | ACM | 연결 서비스와 인증서 리전 |
| DB 비밀번호 보관·자동 교체 | Secrets Manager | 지원 엔진과 교체 구성 |
| 설정값·보안 문자열 보관 | Systems Manager Parameter Store | 자동 교체 요구와 기능 차이 |
| 전용 HSM 제어 필요 | CloudHSM | 키 관리 책임과 운영 부담 |

KMS는 대용량 파일을 직접 계속 암호화하는 저장 서비스가 아니다. 봉투 암호화에서는 데이터 키로 데이터를 암호화하고, KMS 키로 데이터 키를 보호한다. [KMS 개념](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html)

## 4. 보안 서비스 비교

| 문제 | 우선 검토 서비스 |
|---|---|
| HTTP 요청의 SQL 주입·XSS·특정 패턴 차단 | AWS WAF |
| DDoS 방어 | AWS Shield |
| 계정·워크로드의 의심스러운 활동 탐지 | GuardDuty |
| 지원 워크로드의 취약점 검사 | Inspector |
| S3의 민감 데이터 탐지 | Macie |
| 여러 보안 결과와 상태 통합 | Security Hub |

탐지는 차단과 같지 않다. “자동으로 막아야 한다”면 이벤트 이후 대응 흐름까지 필요하다. [보안 서비스 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/security-services.html)

## 5. 다중 계정 설계

Organizations는 계정과 조직 단위(OU)를 묶어 관리한다. SCP로 특정 리전·작업 사용을 제한할 수 있지만 사용자에게 실제 권한을 주는 IAM 정책은 별도로 필요하다. Control Tower는 다중 계정 환경의 초기 구성과 거버넌스에 사용한다. [다중 계정 관리 개요](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/management-governance.html)

## 6. 책임 공유

AWS가 관리하는 범위와 고객이 관리하는 범위를 서비스별로 구분한다. EC2의 게스트 OS 패치는 고객 책임이고, 관리형 DB에서도 데이터·접근 권한·네트워크 구성은 고객이 관리한다. “관리형이므로 보안 설정도 전부 자동”이라는 선택지는 주의한다. [책임 공유 모델](https://aws.amazon.com/compliance/shared-responsibility-model/)

다음: [VPC와 글로벌 네트워크](/en/Study/AWS/Certification/SAA/03-Networking)
