---
title: CI/CD
description: CI/CD 도구와 파이프라인 실습 정리
published: true
date: 2026-09-09T08:00:00.000Z
tags: cicd,devops,jenkins,argocd,gitops
editor: markdown
dateCreated: 2026-08-27T07:01:00.000Z
---

# CI/CD

CI/CD 도구별 개념, 설정 방법, 실습 기록 정리.

## 전체 흐름

```text
Application Repository
        ↓
        CI
├─ Checkout
├─ Build
├─ Test
├─ Image Build
└─ Registry Push
        ↓
GitOps Repository
        ↓
        CD
        ↓
Kubernetes
```

현재 학습 기준으로 보면 아래 조합.

```text
GitHub
  ↓
Jenkins
  ↓
Docker Image
  ↓
ECR
  ↓
GitOps Repository
  ↓
ArgoCD
  ↓
Kubernetes / EKS
```

---

## CI

### Jenkins

- [Jenkins](./CICD/Jenkins)

담당 영역.

```text
Build
Test
Docker Image Build
ECR Push
GitOps Image Tag 변경
```

GitHub Actions는 추후 별도 정리.

---

## CD / GitOps

### ArgoCD

ArgoCD는 한 페이지에 몰아넣지 않고 아래 순서대로 정리.

1. [01. Overview](./CICD/ArgoCD/01-Overview)
2. [02. Architecture](./CICD/ArgoCD/02-Architecture)
3. [03. Install](./CICD/ArgoCD/03-Install)
4. [04. Application](./CICD/ArgoCD/04-Application)
5. [05. Sync](./CICD/ArgoCD/05-Sync)
6. [06. ApplicationSet](./CICD/ArgoCD/06-ApplicationSet)
7. [07. Helm Integration](./CICD/ArgoCD/07-Helm-Integration)
8. [08. Jenkins Integration](./CICD/ArgoCD/08-Jenkins-Integration)
9. [09. Deployment / Rollback](./CICD/ArgoCD/09-Deployment-Rollback)
10. [10. Troubleshooting](./CICD/ArgoCD/10-Troubleshooting)

처음 보면 `01 → 05`까지 먼저 보고,
현재 프로젝트 GitOps 구조를 이해하려면 `06 → 08`까지 이어서 보면 된다.

---

## Jenkins와 ArgoCD 역할 차이

```text
Jenkins
= CI
= Application을 Build하고 배포할 Image 생성

ArgoCD
= CD / GitOps
= Git에 정의된 배포 상태를 Kubernetes에 반영
```

둘을 연결하면.

```text
Developer Push
      ↓
Jenkins
      ↓
ECR Push
      ↓
GitOps Repository 수정
      ↓
ArgoCD
      ↓
EKS
```

ArgoCD가 관리하는 Application에 Jenkins가 다시 `kubectl apply`를 실행하는 식으로 배포 주체를 겹치지 않게 구성.

---

## 정리 방향

플랫폼별로 아래 순서를 기준으로 정리.

```text
01. Overview
02. Architecture
03. 설치 / 기본 설정
기능별 개념
Integration
Troubleshooting
```

개념만 따로 적기보다 실제 CI → GitOps → Kubernetes 흐름 안에서 각 도구가 어디를 담당하는지 같이 기록.