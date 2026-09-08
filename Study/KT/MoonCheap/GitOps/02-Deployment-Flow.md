---
title: GitOps Deployment Flow
description: MoonCheap Jenkins ECR GitOps ArgoCD 배포 흐름 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,gitops,jenkins,ecr,argocd,eks
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# Deployment Flow

## 기본 흐름

```text
Developer
   ↓
GitHub Push / Merge
   ↓
Jenkins
   ├─ Checkout
   ├─ Build
   ├─ Test
   └─ Docker Build
   ↓
Amazon ECR
   ↓
GitOps Repository
Image Tag 변경
   ↓
ArgoCD
   ↓
EKS Deployment Update
```

---

## Branch 기준

### develop

```text
Feature Branch
      ↓ PR
   develop
      ↓
    Jenkins
      ↓
   Dev Image
      ↓
Dev Values Tag 변경
      ↓
    ArgoCD
      ↓
 Dev Environment
```

### main

```text
 develop
    ↓ PR
   main
    ↓
 Jenkins
    ↓
Demo / Production Image
    ↓
Prod Values Tag 변경
    ↓
 ArgoCD
    ↓
Demo / Production
```

현재 기본 정책

- `develop` → Dev 배포 기준
- `main` → Demo / Production 배포 기준
- `hotfix/*` → main 긴급 수정

환경 분리는 최종 합의 후 조정.

---

## Backend 배포 예

Application Commit

```text
a82f91c
```

Jenkins Image Build

```text
123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/moongcheap-backend:a82f91c
```

GitOps Values 변경

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/moongcheap-backend
  tag: a82f91c
```

ArgoCD가 Git 변경 감지.

```text
Old Pod
   ↓ Rolling Update
New Pod : a82f91c
```

---

## Rollback

GitOps Repository의 이전 정상 Commit으로 복구.

```text
Bad Image Tag
     ↓
Git Revert
     ↓
Previous Image Tag
     ↓
ArgoCD Sync
     ↓
Rollback
```

배포 상태 변경을 Git Commit으로 관리하는 것이 핵심.
