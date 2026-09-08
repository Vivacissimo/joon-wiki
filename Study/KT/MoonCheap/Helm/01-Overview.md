---
title: Helm Overview
description: MoonCheap 프로젝트 Helm 구성과 기본 개념 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,helm,kubernetes,eks
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# Helm

## 개념

Kubernetes Manifest를 Template과 Values로 관리하는 도구.

```text
Template
   +
Values
   ↓
Kubernetes Manifest
```

동일한 Kubernetes 구조를 반복 작성하지 않고 값만 변경해서 재사용 가능.

---

## MoonCheap 구성

공통 Chart 1개 사용.

서비스별 설정은 Values로 분리.

```text
charts/
└─ moongcheap-service/
   ├─ Chart.yaml
   ├─ values.yaml
   ├─ templates/
   └─ values/
      ├─ dev/
      └─ prod/
```

서비스별 Values

```text
frontend.yaml
backend.yaml
ai-api.yaml
ai-embedding.yaml
ai-matcher.yaml
ai-llm.yaml
```

---

## 공통 Chart를 사용하는 이유

Frontend / Backend / AI 모두 기본 Kubernetes 구조가 유사.

공통 영역

- Deployment
- Service
- ConfigMap
- Probe
- Resource
- Autoscaling

서비스마다 다른 영역

- Image
- Port
- Resource
- Environment Variable
- Probe Path
- GPU 여부
- Node 배치

```text
공통 Template
     │
     ├─ Frontend Values
     ├─ Backend Values
     └─ AI Values
```

---

## 핵심 관계

```text
Chart.yaml
→ Chart 정보

values.yaml
→ 공통 기본값

values/dev/*.yaml
→ Dev 서비스별 값

values/prod/*.yaml
→ Prod 서비스별 값

templates/*.yaml
→ 실제 Kubernetes Resource Template
```

Helm이 Template과 Values를 합쳐 최종 Manifest 생성.
