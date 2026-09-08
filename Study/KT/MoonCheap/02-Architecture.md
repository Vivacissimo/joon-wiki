---
title: MoonCheap Architecture
description: MoonCheap GitOps 배포 아키텍처 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,architecture,gitops,eks,helm,argocd
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# Architecture

## 배포 구조

```text
GitHub
  ↓
Jenkins
  ↓
Docker Build
  ↓
Amazon ECR
  ↓
GitOps Repository
  ↓
ArgoCD
  ↓
Amazon EKS
```

---

## Application 구성

```text
EKS
├─ Frontend
├─ Backend
└─ AI
   ├─ AI API
   ├─ AI Embedding
   ├─ AI Matcher
   └─ AI LLM
```

Backend

- Spring Boot 모놀리식
- 외부 요청 처리
- AI API 내부 호출
- PostgreSQL 연결

AI API

- Backend에서 내부 호출
- 외부 직접 노출하지 않음

AI LLM

- GPU 필요
- GPU 전용 Node 배치 방향

AI Embedding / Matcher

- CPU Node 배치 방향
- Vector DB 접근 필요 여부 및 방식 확인 필요

---

## GitOps 관점

ArgoCD가 Node를 직접 관리하는 구조가 아님.

```text
GitOps Repository
       ↓
     ArgoCD
       ↓
Helm Manifest
       ↓
 Kubernetes
       ↓
 Deployment
       ↓
     Pod
       ↓
 Scheduler / Karpenter
       ↓
     Node
```

ArgoCD 역할

- Git에 기록된 배포 상태 확인
- 현재 Cluster 상태와 비교
- 차이가 있으면 Sync

Node 배치는 Kubernetes Scheduler와 Karpenter 영역.

---

## Helm 역할

```text
Chart Template
      +
Values
      ↓
Kubernetes Manifest
```

공통 Chart 사용.

서비스별 차이는 Values로 분리.

예

```text
moongcheap-service Chart
        │
        ├─ frontend values
        ├─ backend values
        ├─ ai-api values
        └─ ai-llm values
```

동일한 Deployment Template 사용.

Image, Port, Resource, Probe, Node 배치만 서비스별 설정.
