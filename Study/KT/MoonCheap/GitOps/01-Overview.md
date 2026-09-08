---
title: GitOps Overview
description: MoonCheap 프로젝트에서 사용하는 GitOps 개념 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,gitops,argocd,helm
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# GitOps

## 개념

Git Repository에 원하는 배포 상태를 기록.

ArgoCD가 Git 상태와 Kubernetes Cluster 상태를 비교하고 동기화.

```text
Git
 ↓
Desired State
 ↓
ArgoCD
 ↓
Kubernetes
```

핵심

- Git을 배포 상태의 기준으로 사용
- Kubernetes Manifest 직접 수정 최소화
- 배포 변경 이력 Git으로 추적
- 이전 Commit으로 Rollback 가능

---

## MoonCheap에서의 역할

Application Repository와 GitOps Repository 분리.

```text
Application Repository
        ↓
     Jenkins
        ↓
       ECR
        ↓
GitOps Repository
        ↓
      ArgoCD
        ↓
       EKS
```

Application Repository

- 소스 코드
- Dockerfile
- Jenkins Build 대상

GitOps Repository

- Helm Chart
- 서비스별 Values
- Environment별 Values
- ArgoCD Application 설정

---

## Jenkins와 ArgoCD 역할 분리

Jenkins

- Build
- Test
- Docker Image Build
- ECR Push
- GitOps Image Tag 변경

ArgoCD

- Image Build 하지 않음
- ECR Push 하지 않음
- Git 변경 감지
- Kubernetes 배포 수행

```text
Jenkins = 무엇을 배포할지 Git에 기록
ArgoCD  = Git에 기록된 상태를 Cluster에 반영
```

---

## Image Tag

`latest` 대신 Git Commit SHA 사용.

```text
backend:a82f91c
```

장점

- 어떤 Commit이 배포됐는지 확인 가능
- 배포 버전 추적 가능
- Rollback 기준 명확
