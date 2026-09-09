---
title: 07. ArgoCD Helm Integration
description: ArgoCD에서 Helm Chart와 환경별 values를 사용하는 구조 정리
published: true
date: 2026-09-09T07:56:00.000Z
tags: argocd,cicd,gitops,kubernetes,helm
editor: markdown
dateCreated: 2026-09-09T07:56:00.000Z
---

# 07. ArgoCD Helm Integration

## ArgoCD와 Helm

ArgoCD는 Helm Chart를 읽어서 Kubernetes Manifest를 만든 뒤 배포할 수 있다.

```text
Helm Chart
   +
values.yaml
   ↓
ArgoCD
   ↓
Rendered Manifest
   ↓
Kubernetes
```

여기서 Helm은 **Manifest를 만들어주는 역할**.

실제 배포 상태와 Sync는 ArgoCD가 관리한다.

---

## Helm만 사용할 때

```bash
helm upgrade --install backend ./chart
```

사람이나 CI가 Helm 명령을 직접 실행.

```text
사람 / Jenkins
     ↓
helm upgrade
     ↓
Kubernetes
```

---

## ArgoCD + Helm

```text
Git
├─ Chart
└─ values
    ↓
ArgoCD
    ↓
Kubernetes
```

사람이 매번 `helm upgrade`를 실행하지 않고 Git 변경을 ArgoCD가 반영.

---

## 공통 Chart + 환경별 values

프로젝트처럼 서비스 구조가 비슷하면 공통 Chart 하나와 서비스별 values를 나눌 수 있다.

```text
gitops/
├─ charts/
│  └─ moongcheap-service/
│     ├─ Chart.yaml
│     ├─ values.yaml
│     └─ templates/
│        ├─ deployment.yaml
│        ├─ service.yaml
│        └─ ingress.yaml
│
└─ environments/
   ├─ dev/
   │  ├─ frontend/values.yaml
   │  ├─ backend/values.yaml
   │  └─ ai/values.yaml
   │
   └─ prod/
      ├─ frontend/values.yaml
      ├─ backend/values.yaml
      └─ ai/values.yaml
```

공통 Kubernetes 구조는 Chart에 두고,
서비스별 차이는 values에 둔다.

---

## 기본 values

`charts/moongcheap-service/values.yaml`

```yaml
replicaCount: 1

image:
  repository: ""
  tag: ""

service:
  type: ClusterIP
  port: 80

resources: {}
```

서비스별 values에서 필요한 값만 덮어쓴다.

예.

`environments/dev/backend/values.yaml`

```yaml
replicaCount: 2

image:
  repository: 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/backend
  tag: a82f91c

service:
  port: 8080
```

---

## ArgoCD Application에서 Helm 지정

기본 형태.

```yaml
spec:
  source:
    repoURL: https://github.com/example/gitops.git
    targetRevision: main
    path: charts/moongcheap-service
    helm:
      valueFiles:
        - values.yaml
```

환경별 values 경로를 어떻게 연결할지는 Repository 구성 방식에 따라 정한다.

ArgoCD의 multi-source 기능을 쓰거나 Chart와 values의 위치를 같은 소스 구조 안에서 맞추는 방식도 가능.

핵심은 아래 관계.

```text
공통 Chart
    +
서비스 / 환경별 values
    ↓
최종 Manifest
```

---

## Image Tag 변경 흐름

CI에서 새 Image가 생성되면 Chart Template 자체를 수정할 필요는 없다.

서비스 values의 Image Tag만 변경.

기존.

```yaml
image:
  tag: a82f91c
```

새 Build.

```yaml
image:
  tag: f91ab23
```

흐름.

```text
Jenkins
   ↓
Image Build
   ↓
ECR Push : f91ab23
   ↓
values.yaml tag 변경
   ↓
Git Commit
   ↓
ArgoCD Sync
```

---

## Helm Release와 차이

ArgoCD에서 Helm을 사용할 때 Helm은 주로 **template rendering**에 사용된다.

배포 상태의 기준은 ArgoCD Application과 Git.

따라서 아래처럼 이해.

```text
Helm
= Kubernetes YAML 생성

ArgoCD
= 생성된 리소스의 배포 / Sync / 상태 관리
```

---

## values에 넣는 것

예.

- Replica 수
- Image Repository
- Image Tag
- Service Port
- Ingress 설정
- Resource Request / Limit
- Probe 설정
- HPA 설정
- 환경별 옵션

실제 Secret 값은 values Git에 직접 넣지 않는다.

---

## 프로젝트에서 보는 역할 분리

```text
charts/
= 공통 배포 구조

environments/
= dev / prod + 서비스별 차이

argocd/
= 어떤 Chart와 values를 어느 환경에 배포할지 정의
```

이 세 영역을 분리해두면 서비스별 중복 Manifest가 줄어든다.

---

## 다음

[08. Jenkins Integration](./08-Jenkins-Integration)

---

## 참고

- [Argo CD Helm](https://argo-cd.readthedocs.io/en/stable/user-guide/helm/)
- [Helm Documentation](https://helm.sh/docs/)