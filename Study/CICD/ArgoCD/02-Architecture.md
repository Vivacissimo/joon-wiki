---
title: 02. ArgoCD Architecture
description: ArgoCD 주요 컴포넌트와 동작 구조 정리
published: true
date: 2026-09-09T07:51:00.000Z
tags: argocd,cicd,gitops,kubernetes,architecture
editor: markdown
dateCreated: 2026-09-09T07:51:00.000Z
---

# 02. ArgoCD Architecture

## 전체 구조

ArgoCD는 Git Repository에 정의된 상태와 Kubernetes Cluster 상태를 계속 비교한다.

```text
Git Repository
      │
      ↓
+-----------------------+
|        ArgoCD         |
|                       |
| Repo Server           |
| Application Controller|
| API Server            |
+-----------------------+
      │
      ↓
Kubernetes Cluster
```

---

## 주요 컴포넌트

### API Server

ArgoCD의 외부 진입점.

- Web UI 요청 처리
- CLI 요청 처리
- API 제공
- 인증 / 인가 처리
- Application 조회 / 생성 / Sync 요청 처리

```text
Browser / argocd CLI
          ↓
      API Server
```

---

### Repository Server

Git Repository 내용을 가져와 Kubernetes Manifest를 만들어주는 컴포넌트.

지원 예:

- Plain YAML
- Helm
- Kustomize

```text
Git Repository
      ↓
Repository Server
      ↓
Rendered Manifest
```

Helm을 사용하는 경우 Chart와 values를 읽어서 최종 Manifest를 렌더링한다.

---

### Application Controller

ArgoCD에서 가장 중요한 부분.

Desired State와 Live State를 비교하고 Application 상태를 계산한다.

```text
Desired State (Git)
        │
        ↓
Application Controller
        ↑
        │
Live State (Cluster)
```

담당 작업

- Sync 상태 확인
- Health 상태 확인
- OutOfSync 감지
- Sync 실행
- 자동 Sync 처리
- 리소스 상태 추적

---

### Redis

ArgoCD 내부에서 캐시 용도로 사용.

Repository 정보나 Application 관련 데이터를 빠르게 처리하기 위한 보조 컴포넌트.

직접 배포 흐름을 결정하는 핵심 컴포넌트는 아님.

---

## Application

ArgoCD가 관리하는 배포 단위.

Application 하나에는 크게 아래 정보가 들어간다.

```text
어디서 가져올지
Git Repository / Path / Revision

어디에 배포할지
Cluster / Namespace

어떻게 동기화할지
Sync Policy
```

예:

```text
Application: backend-dev

Source
├─ Repository: gitops repo
├─ Revision: develop
└─ Path: environments/dev/backend

Destination
├─ Cluster: EKS
└─ Namespace: dev
```

Application은 [04-Application](./04-Application)에서 따로 정리.

---

## Kubernetes 안에서 보는 구조

ArgoCD도 Kubernetes 위에서 Pod로 실행된다.

```text
EKS
├─ argocd namespace
│  ├─ argocd-server
│  ├─ argocd-repo-server
│  ├─ argocd-application-controller
│  └─ redis
│
└─ Application namespace
   ├─ frontend
   ├─ backend
   └─ ai
```

ArgoCD 자체와 ArgoCD가 관리하는 Application은 별개.

---

## Git Repository와 Cluster 관계

ArgoCD는 한 Repository에서 여러 Application을 관리할 수 있고,
한 ArgoCD에서 여러 Kubernetes Cluster를 관리할 수도 있다.

```text
                  ┌─ Cluster A
GitOps Repo → ArgoCD
                  └─ Cluster B
```

또는

```text
Repo A ─┐
Repo B ─┼→ ArgoCD → EKS
Repo C ─┘
```

---

## 프로젝트 구조로 보면

```text
MoongCheap GitOps
│
├─ charts/
│  └─ moongcheap-service/
│
├─ environments/
│  ├─ dev/
│  │  ├─ frontend/
│  │  ├─ backend/
│  │  └─ ai/
│  └─ prod/
│     ├─ frontend/
│     ├─ backend/
│     └─ ai/
│
└─ argocd/
   └─ ApplicationSet
```

흐름

```text
GitOps Repository
      ↓
ApplicationSet
      ↓
Application 생성
├─ frontend-dev
├─ backend-dev
├─ ai-dev
├─ frontend-prod
├─ backend-prod
└─ ai-prod
      ↓
EKS
```

ApplicationSet은 Application을 여러 개 반복 작성하지 않기 위해 사용.

---

## 정리

```text
Repo Server
= Git 내용을 읽고 Manifest 생성

Application Controller
= Git 상태와 Cluster 상태 비교 / Sync

API Server
= UI / CLI / API 진입점

Application
= ArgoCD가 관리하는 배포 단위
```

다음: [03. ArgoCD Install](./03-Install)

---

## 참고

- [Argo CD Architecture](https://argo-cd.readthedocs.io/en/stable/operator-manual/architecture/)
- [Argo CD Core Concepts](https://argo-cd.readthedocs.io/en/stable/core_concepts/)