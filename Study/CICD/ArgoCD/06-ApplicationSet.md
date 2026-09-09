---
title: 06. ArgoCD Application 관리 방식
description: Application, ApplicationSet, App of Apps 구조와 차이 정리
published: true
date: 2026-09-09T08:10:00.000Z
tags: argocd,cicd,gitops,kubernetes,application,applicationset,app-of-apps
editor: markdown
dateCreated: 2026-09-09T07:55:00.000Z
---

# 06. ArgoCD Application 관리 방식

ArgoCD에서 여러 서비스를 관리할 때 크게 아래 세 가지 형태로 볼 수 있다.

```text
1. Application
2. ApplicationSet
3. App of Apps
```

셋 다 결국 최종 배포는 `Application`이 담당한다.

차이는 **Application을 어떻게 만들고 묶어서 관리하느냐**에 있다.

---

# 먼저 전체 구조

```text
Application
= Application 하나를 직접 작성

ApplicationSet
= Generator + Template으로 Application 여러 개 생성

App of Apps
= Root Application이 Child Application YAML들을 배포
```

결과적으로는 아래처럼 모두 Application으로 귀결된다.

```text
Application 직접 작성
        ↓
    Application
        ↓
   Kubernetes

ApplicationSet
        ↓
Application 자동 생성
        ↓
   Kubernetes

Root Application
        ↓
Child Application 생성
        ↓
   Kubernetes
```

---

# 1. Application

ArgoCD에서 실제 배포 단위 하나.

Git의 어느 경로를 어떤 Cluster / Namespace에 배포할지 정의한다.

```text
Git Repository
      ↓
 Application
      ↓
Kubernetes
```

예.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-dev
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/example/gitops.git
    targetRevision: main
    path: charts/moongcheap-service

  destination:
    server: https://kubernetes.default.svc
    namespace: dev
```

Application이 보는 핵심 값.

```text
source
= 무엇을 배포할지

repoURL
= 어느 Git Repository인지

targetRevision
= 어느 Branch / Revision인지

path
= Repository 안에서 어느 경로인지

destination
= 어느 Cluster / Namespace에 배포할지
```

## 서비스가 여러 개면

FE / BE / AI를 각각 직접 정의할 수 있다.

```text
frontend-dev.yaml
backend-dev.yaml
ai-dev.yaml
```

prod까지 생기면.

```text
frontend-dev
backend-dev
ai-dev

frontend-prod
backend-prod
ai-prod
```

Application이 6개 필요하다.

### 장점

- 구조가 가장 단순함
- 하나의 Application이 무엇을 배포하는지 바로 보임
- 서비스 수가 적으면 이해하기 편함

### 단점

- 서비스와 환경이 늘어날수록 YAML 반복 증가
- 공통 설정 변경 시 여러 Application 수정 필요

---

# 2. ApplicationSet

여러 `Application`을 자동 생성하는 ArgoCD 리소스.

ApplicationSet 자체가 Deployment를 직접 배포하는 것은 아니다.

```text
ApplicationSet
      ↓
Application 생성
      ↓
Application이 실제 배포
```

예.

```text
ApplicationSet
   ↓
├─ frontend-dev
├─ backend-dev
├─ ai-dev
├─ frontend-prod
├─ backend-prod
└─ ai-prod
```

## 핵심 구성

```text
Generator
= 어떤 Application을 만들지

Template
= 생성되는 Application의 형태
```

예.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: moongcheap-dev
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - service: frontend
          - service: backend
          - service: ai

  template:
    metadata:
      name: '{{service}}-dev'

    spec:
      project: default

      source:
        repoURL: https://github.com/example/gitops.git
        targetRevision: main
        path: charts/moongcheap-service

      destination:
        server: https://kubernetes.default.svc
        namespace: dev
```

생성 결과.

```text
frontend-dev
backend-dev
ai-dev
```

## Generator

대표적으로 아래 형태를 많이 본다.

### List Generator

목록을 직접 작성.

```yaml
- service: frontend
- service: backend
- service: ai
```

### Git Generator

Git Directory / File 구조를 보고 Application 생성.

예.

```text
environments/dev/frontend
environments/dev/backend
environments/dev/ai
```

Directory가 추가되면 해당 구조를 기준으로 Application도 생성하게 만들 수 있다.

### Matrix Generator

여러 Generator를 조합.

예.

```text
서비스
frontend / backend / ai

환경
dev / prod
```

조합 결과.

```text
frontend-dev
backend-dev
ai-dev
frontend-prod
backend-prod
ai-prod
```

## ApplicationSet에서 중요한 점

생성된 Application을 직접 수정하는 방식으로 운영하지 않는다.

```text
공통 Application 설정 변경
→ ApplicationSet Template 수정

서비스별 값 변경
→ Generator 데이터 / values 수정
```

ApplicationSet Controller가 다시 계산하면 생성된 Application은 원본 Template 기준으로 맞춰진다.

### 장점

- Application YAML 반복 감소
- 서비스 / 환경 조합을 자동 생성 가능
- Git Directory나 Cluster 목록을 기준으로 자동 확장 가능
- Multi Cluster / Multi Environment 구조에서 관리 포인트 감소

### 단점

- Generator / Template 구조를 먼저 이해해야 함
- 생성된 Application만 보고 원본 설정 위치를 바로 찾기 어려울 수 있음
- Template을 잘못 수정하면 여러 Application에 동시에 영향

---

# 3. App of Apps

하나의 Root Application이 여러 Child Application을 관리하는 구조.

KT Cloud 글에서는 아래처럼 설명한다.

```text
Root Application
      ↓
Child Applications
      ↓
실제 Kubernetes Resource
```

Root Application은 실제 서비스 Deployment를 직접 가리키는 것이 아니라,
**Child Application YAML이 들어있는 Directory**를 가리킨다.

```text
root-app
   ↓
apps/
├─ frontend.yaml
├─ backend.yaml
└─ ai.yaml
```

각 Child Application이 실제 Helm Chart나 Kubernetes Manifest를 가리킨다.

```text
root-app
   ↓
apps/
├─ frontend Application ─→ frontend resource
├─ backend Application  ─→ backend resource
└─ ai Application       ─→ ai resource
```

## Repository 예시

```text
gitops/
├─ root-app.yaml
│
├─ apps/
│  ├─ frontend.yaml
│  ├─ backend.yaml
│  └─ ai.yaml
│
├─ frontend/
├─ backend/
└─ ai/
```

Root Application 예시.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/example/gitops.git
    targetRevision: main
    path: apps
    directory:
      recurse: true

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

`apps/` 안에는 Child Application 리소스가 위치한다.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-dev
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/example/gitops.git
    targetRevision: main
    path: backend

  destination:
    server: https://kubernetes.default.svc
    namespace: dev
```

## 왜 App of Apps를 쓰는지

ArgoCD에 Application을 하나씩 직접 등록하지 않고 Root Application 하나를 진입점으로 만들 수 있다.

```text
ArgoCD 최초 등록
      ↓
   root-app
      ↓
Child Application들 생성
      ↓
각 Application 배포 시작
```

KT Cloud 글에서도 프로젝트와 환경이 많아질수록 Application 리소스를 하나씩 등록하는 작업과 중복 관리가 커지기 때문에 Root App을 통해 한 곳에서 관리하는 구조를 설명한다.

## 배포 순서

App of Apps의 Child Application들은 기본적으로 서로 독립된 Application이다.

의존 순서가 필요하면 `sync-wave`를 사용할 수 있다.

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

숫자가 낮은 리소스부터 먼저 Sync.

예.

```text
0 : Namespace / CRD
1 : Database 관련 리소스
2 : Backend
3 : Frontend
```

서비스 간 실제 의존성에 맞춰 사용하는 값이지 숫자를 무조건 붙이는 것은 아니다.

## 삭제할 때

App of Apps는 Root / Child Application 관계까지 같이 확인해야 한다.

```text
Root App
   ↓
Child App
   ↓
Workload
```

Git에서 Child Application 정의를 없앴는데 Cluster에 Application이나 Workload가 남는 상황이 생기지 않도록 `prune`, Application 삭제 정책, Finalizer를 같이 확인한다.

Root Application만 삭제하고 Child Application이 남으면 GitOps 관리에서 벗어난 Application이 생길 수 있다.

## 권한

현재 ArgoCD 공식 문서에서는 App of Apps를 관리자 수준의 패턴으로 설명한다.

Root App Repository에 쓰기 권한이 있으면 다른 Application을 생성하고 Project / Namespace 범위까지 건드릴 수 있기 때문.

따라서 Root App 소스 Repository 권한과 PR 검토가 중요하다.

### 장점

- Root Application 하나를 전체 배포 진입점으로 사용 가능
- Child Application 정의가 Git에 명시적으로 보임
- 여러 Application을 트리 형태로 묶어서 관리 가능
- Cluster Bootstrap 구조에 사용 가능

### 단점

- Child Application YAML 자체는 직접 관리해야 해서 반복이 생길 수 있음
- Root / Child 두 단계 Sync 상태를 같이 봐야 함
- 삭제 정책을 잘못 잡으면 Child / Workload가 남을 수 있음
- Root App Repository 권한이 강함

---

# 세 방식 비교

| 구분 | Application | ApplicationSet | App of Apps |
| --- | --- | --- | --- |
| 기본 단위 | Application 1개 | Application 여러 개 자동 생성 | Root App + Child Apps |
| Application 생성 | 직접 작성 | Generator + Template | Root App이 Child YAML 배포 |
| 반복 제거 | 낮음 | 높음 | 중간 |
| 구조 | 평면 | 자동 생성 | 계층형 |
| 서비스 추가 | Application YAML 추가 | Generator 데이터 / Directory 추가 | Child Application YAML 추가 |
| 환경 증가 | YAML 증가 | Generator 조합으로 처리 가능 | Child Application 증가 |
| 관리 포인트 | 각 Application | ApplicationSet | Root + Child Application |
| 처음 이해하기 | 가장 쉬움 | Generator 개념 필요 | Root / Child 개념 필요 |
| 많은 서비스 관리 | 반복이 많아짐 | 잘 맞음 | 중앙 진입점 구성 가능 |

---

# 같은 3개 서비스를 배포하면

## Application 방식

```text
argocd/
├─ frontend.yaml
├─ backend.yaml
└─ ai.yaml
```

```text
frontend.yaml → frontend 배포
backend.yaml  → backend 배포
ai.yaml       → ai 배포
```

세 Application을 각각 작성.

---

## ApplicationSet 방식

```text
argocd/
└─ applicationset.yaml
```

```text
ApplicationSet
      ↓
├─ frontend Application
├─ backend Application
└─ ai Application
```

하나의 Template에서 생성.

---

## App of Apps 방식

```text
argocd/
├─ root-app.yaml
└─ apps/
   ├─ frontend.yaml
   ├─ backend.yaml
   └─ ai.yaml
```

```text
root-app
   ↓
apps/*.yaml
   ↓
├─ frontend Application
├─ backend Application
└─ ai Application
```

Root Application이 Child Application 정의를 배포.

---

# ApplicationSet과 App of Apps 차이

둘 다 여러 Application을 관리한다는 점 때문에 헷갈리기 쉽다.

핵심 차이는 아래처럼 보면 된다.

```text
ApplicationSet
= Application YAML을 Template으로 만들어냄

App of Apps
= Git에 작성된 Application YAML들을 Root App이 배포함
```

## ApplicationSet

```text
Generator
   +
Template
   ↓
Application 생성
```

Application 자체의 반복을 줄이는 쪽에 가깝다.

## App of Apps

```text
Root Application
      ↓
Application YAML Directory
      ↓
Child Application 생성
```

여러 Application을 하나의 진입점 아래 묶는 쪽에 가깝다.

---

# Repository 구조와 연결

KT Cloud 글에서는 App of Apps와 함께 Monorepo / Multirepo, Kustomize base / overlays 구조를 설명한다.

## Monorepo

한 GitOps Repository 안에 앱과 환경 구성을 같이 관리.

```text
gitops/
├─ apps/
├─ environments/
├─ projects/
└─ root-app.yaml
```

## Multirepo

서비스 / 환경 구성을 Repository별로 분리.

```text
frontend-config.git
backend-config.git
dev-environment.git
prod-environment.git
```

이 구조 선택과 ApplicationSet / App of Apps 선택은 완전히 같은 문제는 아니다.

예를 들어 Monorepo에서도 ApplicationSet을 쓸 수 있고 App of Apps도 쓸 수 있다.

```text
Repository 구성
= Git을 어떻게 나눌지

Application 관리 방식
= ArgoCD Application을 어떻게 만들고 묶을지
```

둘을 따로 생각하는 게 이해하기 편하다.

---

# 현재 MoongCheap 구조

현재 잡은 구조.

```text
gitops/
├─ charts/
│  └─ moongcheap-service/
│
├─ environments/
│  ├─ dev/
│  │  ├─ frontend/
│  │  ├─ backend/
│  │  └─ ai/
│  │
│  └─ prod/
│     ├─ frontend/
│     ├─ backend/
│     └─ ai/
│
└─ argocd/
   └─ applicationset.yaml
```

서비스 / 환경 조합.

```text
frontend-dev
backend-dev
ai-dev
frontend-prod
backend-prod
ai-prod
```

이 구조에서는 ApplicationSet으로 반복되는 Application을 생성하는 방식이 자연스럽다.

```text
Git Directory / List
        ↓
ApplicationSet
        ↓
frontend-dev
backend-dev
ai-dev
...
        ↓
공통 Helm Chart + 환경별 values
```

App of Apps로 바꾼다면 구조는 대략 아래처럼 달라진다.

```text
argocd/
├─ root-app.yaml
└─ apps/
   ├─ frontend-dev.yaml
   ├─ backend-dev.yaml
   ├─ ai-dev.yaml
   ├─ frontend-prod.yaml
   ├─ backend-prod.yaml
   └─ ai-prod.yaml
```

즉 현재 프로젝트에서는 **환경 × 서비스 반복을 Template으로 줄이고 싶은 목적**이 더 크기 때문에 ApplicationSet을 사용.

App of Apps의 Root App 계층이 꼭 필요한 구조는 아직 아니다.

---

# 공식 문서 기준으로 같이 볼 부분

현재 ArgoCD 공식 Cluster Bootstrapping 문서에서는 많은 일반적인 경우에 ApplicationSet을 먼저 살펴보는 흐름으로 설명하고, App of Apps는 별도 대안 패턴으로 소개한다.

따라서 아래처럼 이해해두면 된다.

```text
Application
→ 배포 단위의 기본

ApplicationSet
→ 반복되는 Application 자동 생성

App of Apps
→ Application들을 Root 아래 계층적으로 묶는 패턴
```

어느 하나가 항상 상위 방식인 것은 아니다.

서비스 수, 환경 수, Cluster 수, Application 생성 규칙이 얼마나 반복되는지에 따라 선택.

---

# 정리

```text
Application
= 하나의 실제 배포 단위

ApplicationSet
= Application 여러 개를 Generator + Template으로 생성

App of Apps
= Root Application이 Child Application들을 배포하고 묶어서 관리
```

현재 프로젝트.

```text
FE / BE / AI
×
dev / prod

→ 반복 조합이 명확
→ ApplicationSet 사용
```

---

## 다음

[07. Helm Integration](./07-Helm-Integration)

---

## 참고

- [KT Cloud - Kubernetes 환경에서 App of Apps로 구현하는 GitOps 실전 전략](https://tech.ktcloud.com/entry/2025-05-ktcloud-kubernetes-gitops-appofapps-%EA%B5%AC%ED%98%84%ED%99%98%EA%B2%BD-%EC%A0%84%EB%9E%B5)
- [Argo CD - Declarative Setup](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)
- [Argo CD - Cluster Bootstrapping](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)
- [Argo CD - ApplicationSet](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/)
