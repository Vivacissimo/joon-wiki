---
title: Helm Templates
description: MoonCheap Helm templates 파일별 역할 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,helm,kubernetes,templates
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# templates

## 역할

`templates/`는 실제 Kubernetes Resource 구조를 정의하는 영역.

`values.yaml` 값을 읽어서 최종 Manifest 생성.

```text
values
  +
templates
  ↓
Manifest
```

---

## 파일별 역할

### _helpers.tpl

공통 이름과 Label 정의.

- Chart Name
- Release Name
- Kubernetes Label
- ServiceAccount Name

반복 문자열을 함수처럼 재사용.

### deployment.yaml

Application Pod 실행 정의.

주요 설정

- Container Image
- Image Tag
- Replica
- Port
- Environment Variable
- Probe
- CPU / Memory
- GPU
- Node 배치

가장 핵심적인 Template.

### service.yaml

Pod 접근용 Kubernetes Service 생성.

```text
Client Pod
   ↓
Service
   ↓
Application Pod
```

Pod IP가 변경되어도 Service 이름으로 접근.

### configmap.yaml

민감하지 않은 환경변수 저장.

예

```text
SPRING_PROFILES_ACTIVE
AI_API_URL
```

Password / API Key 저장 금지.

### serviceaccount.yaml

Pod가 사용할 ServiceAccount 생성.

IRSA 사용 시 AWS IAM Role과 연결 가능.

```text
Pod
 ↓
ServiceAccount
 ↓
IAM Role
```

### ingress.yaml

외부 요청을 Kubernetes Service로 전달.

```text
Internet
   ↓
ALB / Ingress
   ↓
Service
   ↓
Pod
```

### hpa.yaml

CPU / Memory 기반 Horizontal Pod Autoscaler 생성.

```text
CPU 증가
   ↓
HPA
   ↓
Pod 증가
```

### scaledobject.yaml

KEDA `ScaledObject` 생성.

외부 Metric 기반 Scaling.

MoonCheap에서는 AI LLM Scale-to-Zero 후보.

### pdb.yaml

PodDisruptionBudget 생성.

Node Drain / Upgrade 상황에서 최소 가용 Pod 수 보호.

### servicemonitor.yaml

Prometheus Operator `ServiceMonitor` 생성.

Application Metric 수집 연결.

```text
Application
   ↓
Service
   ↓
ServiceMonitor
   ↓
Prometheus
```

### networkpolicy.yaml

Pod 간 Network 접근 제어.

예

```text
Backend → AI API   허용
Frontend → AI LLM  차단
```

---

## 핵심 흐름

기본 Application 배포

```text
deployment.yaml
      ↓
     Pod

service.yaml
      ↓
Service → Pod
```

외부 공개 시

```text
Ingress
   ↓
Service
   ↓
Pod
```

운영 기능 추가

```text
HPA / KEDA
PDB
ServiceMonitor
NetworkPolicy
```
