---
title: 01. ArgoCD Overview
description: ArgoCD 기본 개념과 GitOps 배포 흐름 정리
published: true
date: 2026-09-09T07:50:00.000Z
tags: argocd,cicd,gitops,kubernetes
editor: markdown
dateCreated: 2026-09-09T07:50:00.000Z
---

# 01. ArgoCD Overview

## ArgoCD

Kubernetes용 GitOps CD 도구.

Git Repository에 적힌 Kubernetes 배포 상태를 읽고 실제 Cluster 상태와 맞춰준다.

```text
Git Repository
      ↓
   ArgoCD
      ↓
Kubernetes Cluster
```

핵심은 **Git을 배포 상태의 기준으로 사용**하는 것.

---

## CI와 ArgoCD 역할

ArgoCD는 보통 CI를 직접 수행하지 않는다.

```text
Application Code
      ↓
Jenkins
├─ Build
├─ Test
├─ Docker Image Build
└─ ECR Push
      ↓
GitOps Repository
      ↓
ArgoCD
      ↓
EKS
```

### Jenkins

- 코드 Build
- Test
- Container Image 생성
- ECR Push
- GitOps Repository의 Image Tag 변경

### ArgoCD

- GitOps Repository 감시
- Git 상태와 Kubernetes 상태 비교
- 변경된 Manifest / Helm 설정 적용
- 배포 상태 확인
- Drift 감지

즉,

```text
Jenkins = 무엇을 배포할지 만든다
ArgoCD = Git에 적힌 상태를 Kubernetes에 반영한다
```

---

## GitOps

배포 설정을 Git으로 관리하는 방식.

기존 방식

```text
CI Server
   ↓
kubectl apply
   ↓
Kubernetes
```

GitOps 방식

```text
CI Server
   ↓
GitOps Repository 수정
   ↓
ArgoCD
   ↓
Kubernetes
```

ArgoCD가 Cluster 안쪽에서 Git을 보고 변경을 가져오는 **Pull 방식**으로 동작한다.

CI Server가 Cluster Credential을 직접 가지고 배포하지 않아도 되는 구조를 만들 수 있다.

---

## Desired State / Live State

ArgoCD에서 계속 비교하는 두 상태.

### Desired State

Git에 적혀 있는 목표 상태.

예:

```yaml
replicaCount: 3
image:
  tag: a82f91c
```

### Live State

실제 Kubernetes Cluster 상태.

```text
Deployment Replica: 2
Image: backend:old-tag
```

두 상태가 다르면 ArgoCD에서 `OutOfSync`로 표시된다.

```text
Git Desired State
       │
       │ compare
       ↓
     ArgoCD
       ↑
       │
Cluster Live State
```

---

## ArgoCD가 관리하는 범위

예:

- Deployment
- Service
- Ingress
- ConfigMap
- HPA
- PDB
- NetworkPolicy
- Helm Chart 기반 리소스
- Custom Resource

Git에 Kubernetes 리소스로 정의되어 있다면 ArgoCD가 관리 가능하다.

---

## ArgoCD가 하지 않는 것

ArgoCD 자체가 아래 작업을 대신하는 것은 아니다.

- Application Compile
- Unit Test
- Docker Image Build
- Container Registry Push
- Terraform으로 VPC/EKS 생성

이런 작업은 Jenkins, GitHub Actions, Terraform 등의 역할.

---

## 프로젝트에서 보는 흐름

현재 구성 기준으로 보면 다음 흐름으로 이해하면 된다.

```text
FE / BE / AI Repository
          ↓
       Jenkins
          ↓
       AWS ECR
          ↓
MoongCheap GitOps Repository
          ↓
        ArgoCD
          ↓
         EKS
```

Application Repository와 배포 Repository를 분리하고,
ArgoCD는 GitOps Repository만 보고 EKS를 관리하는 형태.

---

## 학습 순서

1. [ArgoCD Architecture](./02-Architecture)
2. [ArgoCD Install](./03-Install)
3. [Application](./04-Application)
4. [Sync](./05-Sync)
5. [ApplicationSet](./06-ApplicationSet)
6. [Helm Integration](./07-Helm-Integration)
7. [Jenkins Integration](./08-Jenkins-Integration)
8. [Deployment / Rollback](./09-Deployment-Rollback)
9. [Troubleshooting](./10-Troubleshooting)

---

## 참고

- [Argo CD Documentation](https://argo-cd.readthedocs.io/en/stable/)
- [Argo CD Core Concepts](https://argo-cd.readthedocs.io/en/stable/core_concepts/)