---
title: 03. ArgoCD Install
description: Kubernetes에 ArgoCD 설치하고 접속하는 기본 과정
published: true
date: 2026-09-09T07:52:00.000Z
tags: argocd,cicd,gitops,kubernetes,install
editor: markdown
dateCreated: 2026-09-09T07:52:00.000Z
---

# 03. ArgoCD Install

## 설치 전 확인

필요한 것.

- Kubernetes Cluster
- `kubectl`
- Cluster 접근 권한

현재 Context 확인.

```bash
kubectl config current-context
```

Node 확인.

```bash
kubectl get nodes
```

---

## 1. Namespace 생성

```bash
kubectl create namespace argocd
```

확인.

```bash
kubectl get ns argocd
```

---

## 2. ArgoCD 설치

공식 install manifest 적용.

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Pod 확인.

```bash
kubectl get pods -n argocd
```

대략 아래 컴포넌트가 보인다.

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-applicationset-controller
argocd-dex-server
argocd-redis
```

설치 방식이나 버전에 따라 일부 구성은 다를 수 있음.

---

## 3. Server 확인

```bash
kubectl get svc -n argocd
```

기본 상태에서는 `argocd-server`가 ClusterIP로 생성된다.

외부에서 바로 접근할 수 있는 상태는 아님.

---

## 4. Port Forward

로컬에서 확인할 때.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

접속.

```text
https://localhost:8080
```

개발 단계에서 UI 확인할 때 가장 간단한 방식.

운영 환경에서는 Ingress / Load Balancer 등의 노출 방식을 따로 구성.

---

## 5. 초기 admin Password 확인

```bash
argocd admin initial-password -n argocd
```

또는 Secret에서 확인.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

초기 계정.

```text
username: admin
password: 위에서 조회한 값
```

초기 로그인 후 Password 변경.

---

## 6. ArgoCD CLI 로그인

CLI 설치 후.

```bash
argocd login localhost:8080
```

로컬 Port Forward + 자체 인증서 환경에서는 필요에 따라 아래처럼 접속할 수 있다.

```bash
argocd login localhost:8080 --insecure
```

Application 목록.

```bash
argocd app list
```

---

## 설치 확인 흐름

```text
Kubernetes Cluster
      ↓
argocd Namespace 생성
      ↓
ArgoCD Manifest Apply
      ↓
Pod Running 확인
      ↓
Port Forward
      ↓
Web UI / CLI 접속
```

---

## 프로젝트에서는

EKS 구축 이후 ArgoCD를 올리는 순서로 보면 된다.

```text
Terraform
   ↓
EKS 생성
   ↓
ArgoCD 설치
   ↓
GitOps Repository 연결
   ↓
ApplicationSet 생성
   ↓
FE / BE / AI 배포
```

ArgoCD 설치와 애플리케이션 배포 설정은 분리해서 생각.

---

## 다음

[04. Application](./04-Application)

---

## 참고

- [Argo CD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [Argo CD Installation](https://argo-cd.readthedocs.io/en/stable/operator-manual/installation/)