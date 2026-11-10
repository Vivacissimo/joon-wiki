---
title: SAA 03 — VPC와 글로벌 네트워크
description: 라우팅, NAT, 엔드포인트, 연결 서비스와 글로벌 트래픽 분산
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# VPC와 글로벌 네트워크

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 연결 문제는 경로부터 그린다

VPC는 리전 범위, 서브넷은 하나의 AZ에 속한다. 여러 AZ에 앱을 배치하려면 서브넷도 여러 AZ에 필요하다.

```text
인터넷 → Internet Gateway → 퍼블릭 ALB
                                  ↓
                       프라이빗 앱 서브넷
                          ↓           ↓
                    프라이빗 DB    VPC Endpoint → S3

프라이빗 앱 → Public NAT Gateway → Internet Gateway → 외부 API
```

퍼블릭 서브넷은 Internet Gateway로 향하는 직접 라우트가 있는 서브넷이다. 그 안의 모든 인스턴스가 자동으로 인터넷에 연결되는 것은 아니다. 일반적인 EC2 IPv4 통신에서는 공인 IPv4/EIP, 라우트, SG·NACL 허용도 확인한다. IPv6 경로는 IPv4 NAT와 구분한다. [VPC Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

## 2. SG와 NACL

| 비교 | Security Group | Network ACL |
|---|---|---|
| 적용 | 연결된 리소스의 네트워크 인터페이스 | 서브넷 |
| 규칙 | Allow | Allow·Deny |
| 상태 | Stateful | Stateless |
| 응답 트래픽 | 허용한 연결의 응답을 추적 | 반대 방향 규칙도 필요 |
| 평가 | 허용 규칙 합산 | 낮은 규칙 번호부터 일치 평가 |

ALB → 앱 → DB 구조에서는 앱 SG가 ALB SG를, DB SG가 앱 SG를 소스로 참조하도록 구성할 수 있다. NACL은 응답의 임시 포트도 고려한다. [VPC 보안](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html), [SG와 NACL 비교](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#VPC_Security_Group_NACL_Comparison)

## 3. NAT와 VPC Endpoint

| 요구사항 | 후보 | 주의 |
|---|---|---|
| 프라이빗 EC2에서 외부 인터넷 API 호출 | Public NAT Gateway | 외부에서 먼저 시작하는 연결을 열어 주지 않음 |
| 같은 리전 S3·DynamoDB에 사설 경로로 접근 | Gateway Endpoint | 라우트 테이블 연결과 정책 확인 |
| 지원 AWS 서비스에 사설 IP로 접근 | Interface Endpoint / PrivateLink | DNS·SG·시간 및 데이터 처리 비용 확인 |
| 인터넷에서 웹 앱 접속 | 퍼블릭 ALB 등 | NAT는 인바운드 웹 프록시가 아님 |

일반적인 AZ 단위 NAT 설계에서는 각 AZ의 프라이빗 서브넷이 같은 AZ의 NAT를 사용하도록 구성해 다른 AZ 장애와 교차 AZ 트래픽을 고려한다. 문제에 특정 NAT 배포 방식이 제시되면 그 가용성 범위를 확인한다. [NAT 공식 문서](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)

S3 Gateway Endpoint는 추가 엔드포인트 요금 없이 NAT·IGW 없는 경로를 제공한다. 단, 온프레미스에서 VPN·Direct Connect를 통해 이 Gateway Endpoint를 그대로 이용할 수 있는 것은 아니다. 그런 요구에는 S3 Interface Endpoint 등 다른 경로를 검토한다. Endpoint 정책도 S3 권한을 자동 부여하지 않는다. [S3 엔드포인트 공식 문서](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)

## 4. VPC·온프레미스 연결

| 서비스 | 선택 단서 | 제외할 조건 |
|---|---|---|
| VPC Peering | 소수 VPC 직접 연결 | 전이 라우팅이 필요한 허브 구조 |
| Transit Gateway | 여러 VPC·온프레미스 연결 허브 | 단순한 두 VPC 연결에 무조건 최저 비용은 아님 |
| Site-to-Site VPN | 인터넷을 통한 암호화 터널 | 전용 회선 자체를 요구 |
| Direct Connect | 전용 연결, 일관된 네트워크 성능 | 회선 자체에 암호화가 자동 포함된다고 가정 금지 |
| PrivateLink | 서비스만 사설로 노출 | 전체 네트워크 간 자유로운 연결 목적 |

CIDR 중복과 라우트 전파를 먼저 확인한다. 전용 연결과 암호화는 다른 조건이며, Direct Connect에서 암호화가 필요하면 VPN 조합 또는 지원되는 MACsec 구성을 검토한다. [VPC 연결 옵션](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/introduction.html)

## 5. Route 53 라우팅

| 정책 | 해결하려는 문제 |
|---|---|
| Weighted | 정해진 비율로 DNS 응답 분산 |
| Latency | 사용자에게 지연이 낮은 리전 선택 |
| Failover | 기본 대상 장애 시 보조 대상 |
| Geolocation | 사용자의 지리적 위치 기준 |
| Geoproximity | 리소스 위치와 bias로 분포 조정 |
| Multivalue | 여러 정상 레코드 응답 |

DNS는 연결마다 즉시 트래픽을 옮기는 프록시가 아니다. TTL과 클라이언트 캐시 때문에 장애 전환 시점이 달라질 수 있다. [라우팅 정책](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)

## 6. CloudFront와 Global Accelerator

| 비교 | CloudFront | Global Accelerator |
|---|---|---|
| 중심 목적 | 웹 콘텐츠 배포·캐시 | 글로벌 네트워크 경로와 엔드포인트 가용성 개선 |
| 트래픽 | HTTP·HTTPS | TCP·UDP |
| 캐시 | 지원 | 콘텐츠 캐시 서비스 아님 |
| 선택 단서 | 이미지·다운로드·웹 콘텐츠, 원본 부하 절감 | 고정 진입 IP, 글로벌 TCP·UDP 앱 |

CloudFront는 정적 파일에만 쓰는 서비스가 아니며 동적 웹 콘텐츠에도 사용할 수 있다. 두 서비스 모두 “빠르게 한다”는 설명만 보고 고르지 않는다. [CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html), [Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)

다음: [컴퓨팅과 고가용성](/en/Study/AWS/Certification/SAA/04-Compute-HA)
