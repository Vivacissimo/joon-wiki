---
title: 10. ArgoCD Troubleshooting
description: ArgoCD 배포 문제를 확인하는 순서와 주요 명령 정리
published: true
date: 2026-09-09T07:59:00.000Z
tags: argocd,cicd,gitops,kubernetes,troubleshooting
editor: markdown
dateCreated: 2026-09-09T07:59:00.000Z
---

# 10. ArgoCD Troubleshooting

## 먼저 어디에서 끊겼는지 확인

ArgoCD 문제를 볼 때 처음부터 Pod 로그만 보는 것보다 전체 흐름에서 끊긴 지점을 찾는 게 빠르다.

```text
Application Code
      ↓
Jenkins
      ↓
ECR
      ↓
GitOps Repository
      ↓
ArgoCD
      ↓
Kubernetes
      ↓
Application
```

확인 순서.

```text
1. Jenkins 성공?
2. ECR Image 존재?
3. GitOps Tag 변경?
4. ArgoCD가 변경 감지?
5. Sync 성공?
6. Resource 생성?
7. Pod Ready?
8. Service / Ingress 연결?
```

---

## 기본 명령

Application 목록.

```bash
argocd app list
```

Application 상세.

```bash
argocd app get backend-dev
```

Git과 Cluster 차이.

```bash
argocd app diff backend-dev
```

수동 Sync.

```bash
argocd app sync backend-dev
```

배포 History.

```bash
argocd app history backend-dev
```

---

## OutOfSync가 계속 남을 때

### 1. Git 변경이 실제로 들어갔는지

GitOps Repository 확인.

예.

```yaml
image:
  tag: a82f91c
```

Jenkins가 Tag를 바꿨다고 해도 Push 자체가 실패했을 수 있음.

```text
Jenkins 성공 여부
Git Commit 존재 여부
Git Push 여부
```

순서대로 확인.

---

### 2. targetRevision 확인

Application이 다른 Branch를 보고 있을 수 있다.

```yaml
source:
  targetRevision: main
```

Git은 `develop`을 수정했는데 ArgoCD는 `main`을 보고 있으면 변경이 안 들어옴.

---

### 3. path 확인

```yaml
source:
  path: environments/dev/backend
```

실제 Repository Directory와 다른 경우 Manifest를 못 찾음.

---

### 4. values 경로 확인

Helm 사용 시 자주 확인하는 부분.

- Chart 경로
- values 파일 경로
- 파일 이름
- 상대 경로 기준

Repository 구조를 바꾼 뒤 ApplicationSet 경로를 같이 수정하지 않으면 문제 발생 가능.

---

## Sync Failed

ArgoCD가 Kubernetes에 Resource를 적용하지 못한 상태.

### Namespace 없음

```text
namespace not found
```

Application에 Namespace 자동 생성 옵션이 없다면 먼저 Namespace 생성 필요.

예.

```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
```

또는 Namespace Manifest를 GitOps로 관리.

---

### CRD 없음

Custom Resource를 적용하려는데 해당 CRD가 Cluster에 없는 경우.

예.

```text
Rollout
ServiceMonitor
ExternalSecret
```

먼저 해당 Controller / CRD 설치 여부 확인.

```bash
kubectl get crd
```

---

### 권한 문제

ArgoCD가 Resource를 생성할 권한이 없는 경우.

```text
forbidden
permission denied
```

확인할 것.

- ServiceAccount
- Role / ClusterRole
- RoleBinding / ClusterRoleBinding
- AppProject 제한

---

## Synced인데 Degraded

이 경우 ArgoCD Sync 자체는 성공.

문제는 Kubernetes Application 쪽일 가능성이 큼.

```text
Git
  ↓ 성공
ArgoCD Sync
  ↓ 성공
Kubernetes Resource 생성
  ↓
Pod 문제
```

Pod 확인.

```bash
kubectl get pods -n dev
```

상세.

```bash
kubectl describe pod <pod-name> -n dev
```

로그.

```bash
kubectl logs <pod-name> -n dev
```

---

## ImagePullBackOff

Container Image를 받지 못하는 상태.

확인.

```text
Image Repository 주소
Image Tag 존재 여부
ECR 권한
Node / Pod ECR 접근 권한
imagePullSecrets 사용 여부
```

ECR에 실제 Tag 확인.

GitOps values의 Tag와 ECR Tag가 같은지 확인.

---

## CrashLoopBackOff

Container가 실행됐다가 계속 종료되는 상태.

ArgoCD보다 Application 문제일 가능성이 큼.

확인.

```bash
kubectl logs <pod-name> -n dev --previous
```

주요 원인.

- 환경변수 누락
- Secret 누락
- DB 연결 실패
- Port 설정 오류
- Application 자체 예외

---

## Readiness / Liveness Probe 실패

배포는 됐지만 Health가 정상으로 안 올라올 수 있음.

확인.

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
```

실제 Application Port / Health Path와 Helm values가 같은지 확인.

---

## ApplicationSet에서 Application이 안 생길 때

ApplicationSet 확인.

```bash
kubectl get applicationset -n argocd
```

상세.

```bash
kubectl describe applicationset <name> -n argocd
```

생성된 Application.

```bash
kubectl get applications -n argocd
```

확인할 부분.

- Generator 값
- Git Directory Pattern
- Template 변수 이름
- Repository 접근 권한
- 생성되는 Application 이름 중복

---

## Repository 연결 실패

Private Repository라면 ArgoCD가 Git Repository에 접근할 Credential이 필요.

증상 예.

```text
repository not accessible
authentication required
```

확인.

```bash
argocd repo list
```

Repository 등록 상태와 인증 정보를 확인.

Credential 값 자체는 GitOps Repository에 평문으로 저장하지 않는다.

---

## ArgoCD Pod 자체 확인

ArgoCD 자체가 이상할 때.

```bash
kubectl get pods -n argocd
```

Controller 로그.

```bash
kubectl logs -n argocd \
  deployment/argocd-repo-server
```

Application Controller는 설치 형태에 따라 StatefulSet으로 보일 수 있으므로 실제 리소스 이름 확인 후 로그 확인.

```bash
kubectl get deploy,statefulset -n argocd
```

---

## 자주 헷갈리는 상태

### OutOfSync

```text
Git != Cluster
```

배포 실패라는 뜻은 아님.

---

### Degraded

```text
Resource는 존재하지만 정상 상태 아님
```

Pod나 Deployment 문제 확인.

---

### Missing

Git에는 Resource가 있는데 Cluster에서 찾을 수 없는 상태.

---

### Progressing

새 Deployment가 올라오거나 Pod가 Ready 되는 중.

바로 장애라고 판단하지 않고 진행 상태 확인.

---

## 실전 확인 순서

배포가 안 된 것 같으면 아래 순서로 보면 된다.

```text
[CI]
Jenkins Console Log
      ↓
ECR Image
      ↓

[GitOps]
GitOps Commit
      ↓
values.yaml Image Tag
      ↓

[ArgoCD]
argocd app get
      ↓
argocd app diff
      ↓
Sync / Health
      ↓

[Kubernetes]
kubectl get
      ↓
kubectl describe
      ↓
kubectl logs
```

어느 영역에서 실패했는지 먼저 나눈 뒤 들어가면 된다.

---

## 전체 학습 흐름 다시 보기

1. [Overview](./01-Overview)
2. [Architecture](./02-Architecture)
3. [Install](./03-Install)
4. [Application](./04-Application)
5. [Sync](./05-Sync)
6. [ApplicationSet](./06-ApplicationSet)
7. [Helm Integration](./07-Helm-Integration)
8. [Jenkins Integration](./08-Jenkins-Integration)
9. [Deployment / Rollback](./09-Deployment-Rollback)
10. Troubleshooting

---

## 참고

- [Argo CD FAQ](https://argo-cd.readthedocs.io/en/stable/faq/)
- [Argo CD CLI Commands](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/)
- [Kubernetes Debugging](https://kubernetes.io/docs/tasks/debug/debug-application/)