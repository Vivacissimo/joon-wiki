---
title: 04. ArgoCD Application
description: ArgoCD Application 리소스와 주요 필드 정리
published: true
date: 2026-09-09T07:53:00.000Z
tags: argocd,cicd,gitops,kubernetes,application
editor: markdown
dateCreated: 2026-09-09T07:53:00.000Z
---

# 04. ArgoCD Application

## Application

ArgoCD가 관리하는 **배포 단위**.

Application 하나가 아래 정보를 묶는다.

```text
Source
= 어떤 Git Repository의 어떤 경로를 볼지

Destination
= 어느 Kubernetes Cluster / Namespace에 배포할지

Sync Policy
= 수동 / 자동 동기화 여부
```

---

## 기본 구조

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
    path: environments/dev/backend

  destination:
    server: https://kubernetes.default.svc
    namespace: dev
```

---

## metadata

```yaml
metadata:
  name: backend-dev
  namespace: argocd
```

`name`

- ArgoCD에서 보이는 Application 이름

`namespace`

- Application CR 자체가 생성되는 Namespace
- 보통 `argocd`

Application이 실제로 배포하는 Namespace와는 다름.

---

## project

```yaml
spec:
  project: default
```

ArgoCD의 `AppProject` 지정.

AppProject에서는 Application이 사용할 수 있는 범위를 제어할 수 있다.

예:

- 허용 Git Repository
- 허용 Cluster
- 허용 Namespace
- 허용 Resource 종류

처음에는 `default`를 사용하기도 하고,
환경이 커지면 프로젝트별 AppProject를 따로 나누는 형태.

---

## source

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: environments/dev/backend
```

### repoURL

배포 설정이 있는 Git Repository.

### targetRevision

ArgoCD가 볼 Git Revision.

예:

```text
main
develop
release
v1.0.0
commit SHA
```

### path

Repository 안에서 실제 Manifest / Helm 설정이 있는 위치.

예:

```text
environments/dev/backend
```

---

## destination

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: dev
```

### server

배포 대상 Kubernetes Cluster.

ArgoCD가 자기 자신이 설치된 Cluster를 관리하면 보통 아래 주소 사용.

```text
https://kubernetes.default.svc
```

### namespace

Application 리소스가 실제 배포될 Namespace.

예:

```text
dev
prod
```

---

## syncPolicy

자동 Sync를 사용하는 예.

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

`prune`

- Git에서 삭제된 리소스를 Cluster에서도 삭제

`selfHeal`

- Cluster에서 수동으로 바뀐 값을 Git 상태로 되돌림

자세한 내용은 [05-Sync](./05-Sync).

---

## Helm을 사용하는 경우

Application에서 Chart와 values 위치를 지정할 수 있다.

예:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: charts/moongcheap-service
  helm:
    valueFiles:
      - ../../environments/dev/backend/values.yaml
```

실제 상대 경로 사용 가능 여부와 기준은 Repository 구조 및 ArgoCD 설정 방식에 따라 확인 필요.

프로젝트에서는 Chart와 환경별 values를 나누고 Application/ApplicationSet에서 연결하는 구조로 이해하면 된다.

---

## Sync Status

ArgoCD에서 Git과 Cluster 상태가 같은지 보여주는 값.

### Synced

Git 상태와 실제 Cluster 상태가 같음.

### OutOfSync

둘이 다름.

예:

```text
Git: replicaCount = 3
Cluster: replicaCount = 2
```

→ `OutOfSync`

---

## Health Status

배포된 리소스가 정상 상태인지 보여주는 값.

대표 상태.

```text
Healthy
Progressing
Degraded
Missing
Unknown
```

Sync와 Health는 다른 개념.

```text
Synced + Healthy
= Git 상태도 같고 실제 서비스도 정상

Synced + Degraded
= Git 반영은 됐지만 Pod 등이 정상 아님

OutOfSync + Healthy
= 서비스는 동작하지만 Git과 실제 상태가 다름
```

---

## 프로젝트에서 Application을 직접 여러 개 만들면

```text
frontend-dev
backend-dev
ai-dev
frontend-prod
backend-prod
ai-prod
```

서비스와 환경이 늘수록 Application YAML이 반복된다.

그래서 다음 단계에서 `ApplicationSet` 사용.

---

## 다음

[05. Sync](./05-Sync)

---

## 참고

- [Application Specification](https://argo-cd.readthedocs.io/en/stable/user-guide/application-specification/)
- [Projects](https://argo-cd.readthedocs.io/en/stable/user-guide/projects/)