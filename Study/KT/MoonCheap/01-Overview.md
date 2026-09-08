---
title: MoonCheap GitOps Overview
description: MoonCheap 프로젝트의 GitOps 및 Kubernetes 배포 구성 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,gitops,helm,argocd,jenkins,eks
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# MoonCheap

## 프로젝트 배포 구성

MoonCheap 애플리케이션을 AWS EKS에 배포하기 위한 GitOps 구성.

주요 대상

- Frontend: Next.js
- Backend: Spring Boot
- AI API: FastAPI
- AI LLM
- AI Embedding
- AI Matcher

주요 도구

- Jenkins
- Amazon ECR
- Helm
- ArgoCD
- Amazon EKS
- KEDA
- Karpenter
- Prometheus / Grafana

---

## 전체 흐름

```text
Application Repository
        ↓
     Jenkins
 Build / Test
        ↓
 Docker Image Build
        ↓
       ECR
        ↓
GitOps Repository
 Helm Values 변경
        ↓
      ArgoCD
        ↓
       EKS
```

핵심

- Jenkins: CI 및 Image Build
- ECR: Container Image 저장
- GitOps Repository: 배포 상태 관리
- Helm: Kubernetes Manifest 생성
- ArgoCD: Git 상태와 EKS 상태 동기화
- EKS: 실제 Application 실행

---

## Repository 역할

```text
Application Repo
├─ Frontend
├─ Backend
└─ AI

GitOps Repo
├─ Helm Chart
├─ Environment Values
└─ ArgoCD 설정
```

Application Repository

- 소스 코드 관리
- Dockerfile 관리
- Build / Test

GitOps Repository

- Image Tag 관리
- Helm Values 관리
- Kubernetes 배포 상태 관리

---

## 현재 기준

확정

- Jenkins 기반 CI/CD
- ECR Image Push
- ArgoCD 기반 배포
- EKS Application 배포
- Image Tag는 Git Commit SHA 사용
- AI API는 외부 비노출
- AI LLM은 GPU Node 배치 방향

추가 확인 필요

- Dev / Demo(Prod) 환경 최종 구성
- HPA / KEDA 대상 및 Metric
- Karpenter GPU Scale-to-Zero 조건
- Secret Store 최종 방식
- NetworkPolicy 세부 정책
- Resource Request / Limit 실측값
