---
title: Karpenter
description: EKS 노드 오토스케일링을 위한 Karpenter 정리
published: true
date: 2026-09-17T14:18:00.000Z
tags: aws, eks, karpenter, kubernetes, autoscaling
editor: markdown
dateCreated: 2026-09-17T14:18:00.000Z
---

# Karpenter

Karpenter는 Kubernetes에서 **필요한 노드를 자동으로 생성하고 제거하는 Node Autoscaler**이다.

AWS EKS에서 Pod가 실행될 자원이 부족할 때 필요한 EC2 인스턴스를 생성하고, 사용하지 않는 노드는 정리한다.

```text
Pod 실행 공간 부족
        ↓
Pod Pending
        ↓
Karpenter가 감지
        ↓
Pod 요구사항 확인
        ↓
적절한 EC2 인스턴스 생성
        ↓
새 Node가 EKS에 참가
        ↓
Pending Pod 스케줄링
```

<br>

## 1. HPA와 Karpenter

HPA와 Karpenter는 스케일링 대상이 다르다.

| 구분 | HPA | Karpenter |
|---|---|---|
| 대상 | Pod | Node |
| 역할 | Pod 개수 증가/감소 | EC2 Node 증가/감소 |
| 기준 | CPU, Memory, Custom Metric 등 | Pending Pod의 리소스 요구사항 |
| 동작 결과 | Deployment의 Replica 변경 | 새로운 EC2 생성 또는 기존 Node 제거 |

예를 들어 Backend Pod가 3개에서 10개로 증가했는데 기존 Node에 공간이 없다면 다음과 같이 동작한다.

```text
트래픽 증가
   ↓
HPA
   ↓
Backend Pod
3개 → 10개
   ↓
기존 Node 자원 부족
   ↓
일부 Pod Pending
   ↓
Karpenter
   ↓
EC2 Node 추가
   ↓
Pending Pod 배치
```

정리하면 다음과 같다.

> HPA는 Pod를 늘리고, Karpenter는 그 Pod가 실행될 Node를 늘린다.

<br>

## 2. 동작 방식

Karpenter는 단순히 정해진 Node Group의 수를 늘리는 방식이 아니다.

Pending 상태의 Pod를 확인하고 해당 Pod가 필요로 하는 조건을 기준으로 적절한 EC2 인스턴스를 선택한다.

확인하는 조건의 예시는 다음과 같다.

- CPU
- Memory
- CPU Architecture
- Availability Zone
- Instance Family
- Spot / On-Demand
- NodeSelector
- Taint / Toleration

```text
Pending Pod

CPU: 2
Memory: 4Gi
Architecture: amd64
Zone: ap-northeast-2a

        ↓

Karpenter

        ↓

조건에 맞는 EC2 인스턴스 선택

        ↓

Node 생성
```

<br>

## 3. Cluster Autoscaler와 차이

기존 EKS에서는 Cluster Autoscaler를 사용해 Node 수를 조절하는 방식도 많이 사용한다.

### Cluster Autoscaler

미리 생성한 Auto Scaling Group 또는 Managed Node Group의 크기를 조절한다.

```text
EKS
└── Node Group
    └── t3.large
        ├── Node
        ├── Node
        └── Node 추가
```

즉, 이미 정해진 Node Group 안에서 Node 수를 늘리고 줄인다.

### Karpenter

Pod의 요구사항을 기준으로 필요한 EC2 인스턴스를 직접 선택해 생성할 수 있다.

```text
Pod Pending
    ↓
Karpenter
    ↓
필요한 CPU / Memory / Zone 확인
    ↓
조건에 맞는 EC2 선택
    ↓
Node 생성
```

| 구분 | Cluster Autoscaler | Karpenter |
|---|---|---|
| 확장 기준 | 기존 Node Group | Pending Pod 요구사항 |
| 인스턴스 선택 | Node Group에 미리 정의 | 조건에 맞게 동적 선택 가능 |
| Node Group 의존성 | 높음 | 낮음 |
| AWS EC2 선택 유연성 | 상대적으로 낮음 | 높음 |

<br>

## 4. 주요 구성

Karpenter에서 주로 확인할 리소스는 다음 두 가지이다.

```text
Karpenter
├── NodePool
└── EC2NodeClass
```

### NodePool

어떤 조건의 Node를 생성할 수 있는지 정의한다.

예를 들어 다음과 같은 조건을 설정할 수 있다.

- amd64 / arm64
- Spot / On-Demand
- Instance Category
- Instance Family
- Availability Zone
- Node 개수 및 리소스 제한

예시:

```yaml
requirements:
  - key: kubernetes.io/arch
    operator: In
    values:
      - amd64

  - key: karpenter.sh/capacity-type
    operator: In
    values:
      - spot
      - on-demand
```

### EC2NodeClass

AWS에서 실제 EC2 Node를 생성할 때 필요한 설정을 정의한다.

주로 다음 내용을 설정한다.

- AMI
- IAM Role
- Subnet
- Security Group
- EBS
- User Data

역할을 나누면 다음과 같다.

```text
NodePool
= 어떤 Node를 만들 것인가

EC2NodeClass
= AWS에서 그 Node를 어떻게 만들 것인가
```

<br>

## 5. EKS에서의 전체 흐름

```text
사용자 요청 증가
      ↓
Application 트래픽 증가
      ↓
HPA
      ↓
Pod 증가
      ↓
Node 리소스 부족
      ↓
Pod Pending
      ↓
Karpenter
      ↓
EC2 Instance 생성
      ↓
EKS Worker Node 참가
      ↓
Pending Pod 배치
```

트래픽이 줄어들면 Pod 수가 줄고, 불필요해진 Node는 Karpenter가 정리할 수 있다.

```text
트래픽 감소
   ↓
Pod 감소
   ↓
Node 사용량 감소
   ↓
Pod 재배치
   ↓
불필요 Node 제거
```

<br>

## 6. 정리

Karpenter는 EKS에서 **Node 단위의 자동 확장과 축소를 담당한다.**

HPA가 Pod 수를 늘렸는데 기존 Node에 자리가 없다면 Karpenter가 새로운 EC2 Node를 생성한다.

기존 Node Group의 크기만 조절하는 방식보다 Pod 요구사항에 맞춰 인스턴스를 선택할 수 있어 EC2 구성에 대한 유연성이 높다.

```text
HPA
= Pod Scaling

Karpenter
= Node Scaling
```

EKS에서 애플리케이션 오토스케일링을 구성할 때는 두 기능을 함께 이해하는 것이 중요하다.
