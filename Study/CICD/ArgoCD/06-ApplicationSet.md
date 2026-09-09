---
title: 06. ArgoCD ApplicationSet
description: 여러 ArgoCD Application을 반복 생성하는 ApplicationSet 정리
published: true
date: 2026-09-09T07:55:00.000Z
tags: argocd,cicd,gitops,kubernetes,applicationset
editor: markdown
dateCreated: 2026-09-09T07:55:00.000Z
---

# 06. ArgoCD ApplicationSet

## ApplicationSet

여러 ArgoCD `Application`을 반복해서 만들기 위한 리소스.

서비스와 환경이 늘어나면 Application YAML도 계속 늘어난다.

예.

```text
frontend-dev
backend-dev
ai-dev
frontend-prod
backend-prod
ai-prod
```

Application을 6개 직접 작성하는 대신 ApplicationSet 하나에서 생성 가능.

---

## Application과 차이

### Application

실제 배포 단위 하나.

```text
backend-dev
```

### ApplicationSet

Application 여러 개를 생성하는 템플릿.

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

ApplicationSet이 직접 Deployment를 배포하는 것이 아니라,
Application을 만든 뒤 각 Application이 실제 배포를 관리한다.

---

## 기본 구성

ApplicationSet은 크게 두 부분으로 보면 된다.

```text
Generator
= 어떤 Application들을 만들지

Template
= 생성되는 Application 모양
```

---

## List Generator 예시

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

결과.

```text
frontend-dev
backend-dev
ai-dev
```

---

## Generator 종류

대표적으로 아래 방식이 있다.

### List Generator

직접 목록 작성.

```yaml
- service: frontend
- service: backend
- service: ai
```

서비스 수가 적고 구조가 명확할 때 이해하기 쉬움.

---

### Git Generator

Git Repository의 Directory나 File 구조를 보고 Application 생성.

예.

```text
environments/dev/frontend
environments/dev/backend
environments/dev/ai
```

Directory 구조를 읽어서 자동으로 Application 생성 가능.

서비스가 추가될 때 Directory만 추가해도 Application 생성 흐름을 만들 수 있다.

---

### Matrix Generator

여러 Generator 조합.

예.

```text
서비스
frontend / backend / ai

환경
dev / prod
```

두 목록을 조합.

```text
frontend-dev
backend-dev
ai-dev
frontend-prod
backend-prod
ai-prod
```

환경과 서비스 조합이 많아질 때 사용하기 좋다.

---

## 프로젝트 구조와 연결

현재 GitOps 구조를 기준으로 보면 아래처럼 연결 가능.

```text
gitops/
├─ charts/
│  └─ moongcheap-service/
│
├─ environments/
│  ├─ dev/
│  │  ├─ frontend/
│  │  │  └─ values.yaml
│  │  ├─ backend/
│  │  │  └─ values.yaml
│  │  └─ ai/
│  │     └─ values.yaml
│  │
│  └─ prod/
│     ├─ frontend/
│     ├─ backend/
│     └─ ai/
│
└─ argocd/
   └─ applicationset.yaml
```

ApplicationSet에서는 서비스 이름과 환경 정보를 이용해 공통 Chart에 각 values를 연결.

```text
ApplicationSet
      ↓
frontend-dev ─→ common chart + frontend dev values
backend-dev  ─→ common chart + backend dev values
ai-dev       ─→ common chart + ai dev values
```

---

## 왜 ApplicationSet을 쓰는지

Application을 직접 관리하면 서비스가 늘어날수록 같은 코드가 반복된다.

```text
Application 1
Application 2
Application 3
Application 4
...
```

ApplicationSet을 쓰면 반복되는 부분을 Template으로 묶는다.

```text
Generator + Template
       ↓
Application 자동 생성
```

수정 포인트도 줄어든다.

---

## 주의할 부분

ApplicationSet에서 생성된 Application은 원본 ApplicationSet의 Template 영향을 받는다.

생성된 Application만 직접 수정하면 이후 다시 ApplicationSet 값으로 돌아갈 수 있다.

수정할 내용이 공통 설정이면 ApplicationSet 수정.

서비스별 설정이면 values나 Generator 데이터 수정.

---

## 정리

```text
Application
= 서비스 하나의 배포 상태 관리

ApplicationSet
= Application 여러 개를 생성/관리
```

현재처럼 FE / BE / AI와 dev / prod 조합이 있는 구조에서는 ApplicationSet으로 묶는 이유가 명확하다.

---

## 다음

[07. Helm Integration](./07-Helm-Integration)

---

## 참고

- [ApplicationSet](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/)
- [ApplicationSet Generators](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators/)