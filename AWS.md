---
title: AWS
description: 
published: true
date: 2026-06-08T06:31:16.316Z
tags: 
editor: markdown
dateCreated: 2026-06-08T06:31:16.316Z
---

# Amazon Web Services?
<br>

## 1. AWS란?

**AWS**는 **Amazon Web Services**의 약자이다.

AWS는 Amazon에서 제공하는 **클라우드 컴퓨팅 플랫폼**이다.  
사용자가 직접 서버실, 물리 서버, 네트워크 장비, 스토리지를 구매하고 운영하지 않아도, 인터넷을 통해 필요한 IT 자원을 빌려서 사용할 수 있게 해주는 서비스이다.

AWS에서 사용할 수 있는 대표적인 자원은 아래와 같다.

- 서버
- 네트워크
- 스토리지
- 데이터베이스
- 컨테이너 실행 환경
- 모니터링
- 보안 및 권한 관리
- 배포 자동화 도구

참고: [AWS 공식 문서 - What is AWS?](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html)

---

## 2. AWS를 사용하는 이유

기존에는 서비스를 운영하려면 직접 물리 서버를 구매하고, 네트워크를 구성하고, 데이터센터나 서버실을 관리해야 했다.

AWS를 사용하면 필요한 만큼만 서버와 인프라를 생성해서 사용할 수 있다.

```text
기존 방식
서버 구매 → 설치 → 네트워크 구성 → 운영 → 장애 대응 → 확장

AWS 방식
필요한 리소스 선택 → 생성 → 사용 → 필요 없으면 삭제
```

AWS를 사용하는 대표적인 이유는 아래와 같다.

| 이유 | 설명 |
|---|---|
| 빠른 구축 | 서버나 네트워크를 직접 구매하지 않고 빠르게 생성 가능 |
| 확장성 | 트래픽 증가에 따라 서버 수를 늘리거나 줄일 수 있음 |
| 비용 효율성 | 사용한 만큼 비용을 지불하는 구조 |
| 관리 편의성 | 물리 장비 관리 부담 감소 |
| 다양한 서비스 | 서버, DB, 스토리지, 보안, 모니터링 등 다양한 서비스 제공 |
| 글로벌 인프라 | 여러 국가와 지역에 인프라를 배포 가능 |

---

## 3. 클라우드 컴퓨팅이란?

**클라우드 컴퓨팅**은 인터넷을 통해 서버, 스토리지, 데이터베이스, 네트워크 같은 IT 리소스를 제공받아 사용하는 방식이다.

내 컴퓨터나 회사 서버실에 모든 장비를 직접 두는 것이 아니라, 클라우드 제공자가 운영하는 인프라를 빌려 쓰는 구조이다.

```text
사용자
  |
  | 인터넷
  v
AWS Cloud
  |
  ├── Server
  ├── Storage
  ├── Database
  ├── Network
  └── Security
```

---

## 4. 온프레미스와 AWS의 차이

**온프레미스**는 회사가 직접 서버와 네트워크 장비를 구매해서 운영하는 방식이다.

**AWS**는 Amazon이 운영하는 클라우드 인프라를 필요한 만큼 빌려 쓰는 방식이다.

| 구분 | 온프레미스 | AWS |
|---|---|---|
| 서버 구매 | 직접 구매 | 필요할 때 생성 |
| 초기 비용 | 높음 | 낮음 |
| 확장 | 장비 추가 필요 | 설정으로 확장 가능 |
| 운영 관리 | 직접 관리 | 일부 영역은 AWS가 관리 |
| 장애 대응 | 직접 구성 필요 | AWS 서비스 활용 가능 |
| 비용 구조 | 장비 구매 중심 | 사용량 기반 과금 |

---

## 5. AWS의 기본 개념

AWS를 이해할 때는 아래 개념을 먼저 잡는 것이 좋다.

| 개념 | 설명 |
|---|---|
| Region | AWS 인프라가 위치한 지리적 지역 |
| Availability Zone | 하나의 Region 안에 있는 독립적인 데이터센터 단위 |
| VPC | AWS 안에서 사용하는 가상 네트워크 |
| EC2 | AWS에서 생성하는 가상 서버 |
| EBS | EC2에 붙이는 가상 디스크 |
| S3 | 객체 스토리지 서비스 |
| IAM | 사용자, 권한, 역할을 관리하는 서비스 |
| Load Balancer | 트래픽을 여러 서버로 분산하는 서비스 |
| CloudWatch | 로그와 메트릭을 수집하고 모니터링하는 서비스 |

---

## 6. AWS 서비스의 큰 분류

AWS에는 많은 서비스가 있지만, 처음에는 아래 분야로 나누어 이해하면 좋다.

```text
AWS
├── Compute
├── Networking
├── Storage
├── Database
├── Security
├── Monitoring
└── Cost Management
```

각 분야별 대표 서비스는 아래와 같다.

| 분야 | 대표 서비스 | 설명 |
|---|---|---|
| Compute | EC2, Lambda | 서버 또는 코드 실행 환경 |
| Networking | VPC, Load Balancer, Route 53 | 네트워크와 트래픽 관리 |
| Storage | EBS, S3, EFS | 디스크, 파일, 객체 저장소 |
| Database | RDS, DynamoDB | 관계형/비관계형 데이터베이스 |
| Security | IAM, KMS, Secrets Manager | 권한, 암호화, 비밀값 관리 |
| Monitoring | CloudWatch, CloudTrail | 로그, 메트릭, 감사 기록 |
| Cost Management | Billing, Cost Explorer, Budgets | 비용 확인과 예산 관리 |

---

## 7. AWS의 사용 방식

AWS는 보통 아래 방식으로 사용할 수 있다.

| 방식 | 설명 |
|---|---|
| AWS Management Console | 웹 브라우저에서 AWS를 관리하는 화면 |
| AWS CLI | 터미널에서 명령어로 AWS를 관리하는 도구 |
| SDK | Python, Java, Go 같은 코드에서 AWS를 제어하는 방식 |
| IaC | Terraform, CloudFormation 등 코드로 인프라를 관리하는 방식 |

예를 들어 EC2 서버를 만들 때도 아래처럼 여러 방식이 가능하다.

```text
1. AWS Console에서 직접 생성
2. AWS CLI 명령어로 생성
3. Terraform 코드로 생성
4. CloudFormation 템플릿으로 생성
```

---

## 8. AWS의 과금 방식

AWS는 대부분 **사용량 기반 과금** 구조이다.

리소스를 얼마나 오래 사용했는지, 얼마나 많은 데이터를 저장했는지, 얼마나 많은 트래픽이 발생했는지에 따라 비용이 달라진다.

예시는 아래와 같다.

| 리소스 | 과금 기준 |
|---|---|
| EC2 | 인스턴스 타입과 실행 시간 |
| EBS | 디스크 용량과 사용 기간 |
| S3 | 저장 용량, 요청 수, 데이터 전송량 |
| Load Balancer | 실행 시간과 처리량 |
| NAT Gateway | 실행 시간과 데이터 처리량 |
| EKS | 클러스터 실행 시간 |

AWS를 사용할 때는 리소스를 생성하는 것뿐만 아니라, 사용하지 않는 리소스를 정리하는 것도 중요하다.

---

## 9. AWS의 장점

AWS의 대표적인 장점은 아래와 같다.

| 장점 | 설명 |
|---|---|
| 빠른 인프라 구성 | 몇 분 안에 서버, 네트워크, 스토리지 생성 가능 |
| 확장성 | 트래픽 증가에 따라 리소스 확장 가능 |
| 글로벌 서비스 | 여러 Region을 활용해 전 세계 서비스 가능 |
| 다양한 관리형 서비스 | DB, Kubernetes, 모니터링 등을 관리형으로 사용 가능 |
| 자동화 친화적 | CLI, API, Terraform 등으로 자동화 가능 |
| 비용 제어 가능 | 사용량 기반으로 비용 확인 및 제한 가능 |

---

## 10. AWS 사용 시 주의할 점

AWS는 편리하지만, 잘못 사용하면 비용이나 보안 문제가 생길 수 있다.

주의할 점은 아래와 같다.

| 항목 | 설명 |
|---|---|
| 비용 관리 | 사용하지 않는 리소스를 삭제하지 않으면 비용 발생 |
| 권한 관리 | IAM 권한을 과하게 주면 보안 위험 증가 |
| 네트워크 설정 | Security Group, Subnet, Route Table 이해 필요 |
| 데이터 보호 | 백업, 암호화, 접근 제어 필요 |
| 모니터링 | 장애 확인을 위해 로그와 메트릭 수집 필요 |

---

## 11. AWS를 한 문장으로 정리

AWS는 서버, 네트워크, 스토리지, 데이터베이스 같은 IT 인프라를 직접 구매하지 않고, 인터넷을 통해 필요한 만큼 빌려서 사용할 수 있게 해주는 클라우드 플랫폼이다.

```text
AWS = 필요한 IT 인프라를 원하는 만큼 빌려 쓰는 클라우드 플랫폼
```

---

## 12. 참고 자료

- [AWS 공식 문서 - What is AWS?](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html)
- [AWS 공식 사이트](https://aws.amazon.com/)
- [AWS Cloud Computing 개요](https://aws.amazon.com/what-is-cloud-computing/)
- [AWS 글로벌 인프라](https://aws.amazon.com/about-aws/global-infrastructure/)