---
title: 05. ArgoCD Sync
description: ArgoCD Sync 상태와 자동 동기화 동작 정리
published: true
date: 2026-09-09T07:54:00.000Z
tags: argocd,cicd,gitops,kubernetes,sync
editor: markdown
dateCreated: 2026-09-09T07:54:00.000Z
---

# 05. ArgoCD Sync

## Sync

Git의 Desired State를 Kubernetes Cluster에 반영하는 작업.

```text
Git Desired State
       ↓
     ArgoCD
       ↓
Kubernetes Live State
```

두 상태가 같아지면 `Synced`.

다르면 `OutOfSync`.

---

## 수동 Sync

자동 Sync를 켜지 않은 경우 사람이 직접 실행.

CLI.

```bash
argocd app sync backend-dev
```

UI에서는 Application에서 `SYNC` 실행.

흐름.

```text
Git 변경
   ↓
OutOfSync
   ↓
사람이 Sync 실행
   ↓
Cluster 반영
   ↓
Synced
```

---

## 자동 Sync

Application에 자동 동기화 설정.

```yaml
syncPolicy:
  automated: {}
```

Git에서 변경이 감지되면 ArgoCD가 Cluster에 자동 반영.

```text
Git Commit
   ↓
ArgoCD 감지
   ↓
자동 Sync
   ↓
Kubernetes 반영
```

---

## prune

Git에서 리소스를 삭제했을 때 Cluster에서도 삭제할지 결정.

```yaml
syncPolicy:
  automated:
    prune: true
```

예.

기존 Git.

```text
Deployment
Service
ConfigMap
```

ConfigMap 삭제 후.

```text
Deployment
Service
```

`prune: true`면 Cluster의 ConfigMap도 제거.

`prune`이 없으면 Git에서는 없어졌는데 Cluster에는 남아 있을 수 있음.

---

## selfHeal

Cluster에서 직접 수정된 내용을 Git 상태로 되돌리는 기능.

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

예.

Git.

```yaml
replicaCount: 2
```

사람이 직접 변경.

```bash
kubectl scale deployment backend --replicas=5
```

상태.

```text
Git: 2
Cluster: 5
```

ArgoCD가 차이를 감지하고 다시 2로 맞춤.

```text
kubectl 수동 수정
      ↓
OutOfSync
      ↓
selfHeal
      ↓
Git 상태로 복구
```

GitOps에서는 Cluster를 직접 수정하지 않고 Git을 수정하는 이유가 여기 있음.

---

## 자동 Sync 예시

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

의미.

```text
Git 변경 → 자동 배포
Git 삭제 → Cluster 리소스 삭제
Cluster 수동 변경 → Git 상태로 복구
```

---

## Sync와 Health 차이

### Sync

Git 상태와 Cluster 상태가 같은지 확인.

```text
Synced
OutOfSync
```

### Health

실제 Kubernetes Resource가 정상인지 확인.

```text
Healthy
Progressing
Degraded
```

예.

```text
Synced + Healthy
```

정상적인 상태.

```text
Synced + Degraded
```

Git 설정은 반영됐지만 Pod가 CrashLoopBackOff 같은 상태일 수 있음.

---

## Drift

Git과 실제 Cluster 상태가 달라진 상태.

예.

```text
Git
replicas: 2

Cluster
replicas: 5
```

사람이 직접 `kubectl edit`, `kubectl patch`, `kubectl scale` 등을 사용하면 발생 가능.

ArgoCD는 이런 차이를 감지한다.

---

## 운영 흐름

GitOps에서는 아래 흐름을 기준으로 잡는다.

```text
잘못된 흐름

kubectl edit
   ↓
Cluster 직접 변경


기본 흐름

Git 수정
   ↓
Commit / Merge
   ↓
ArgoCD Sync
   ↓
Cluster 변경
```

긴급 수정으로 Cluster를 직접 건드렸다면 이후 Git에도 같은 변경을 반영하지 않으면 다시 차이가 발생한다.

---

## Sync 상태 확인

```bash
argocd app get backend-dev
```

Application 전체 목록.

```bash
argocd app list
```

차이 확인.

```bash
argocd app diff backend-dev
```

수동 Sync.

```bash
argocd app sync backend-dev
```

---

## 프로젝트에서

Dev 환경은 자동 Sync를 적용하기 편한 편.

```text
develop Merge
      ↓
Jenkins
      ↓
ECR Push
      ↓
GitOps dev values Image Tag 변경
      ↓
ArgoCD Auto Sync
      ↓
Dev EKS 배포
```

Demo / Prod는 자동화 범위를 별도로 정해서 운영 가능.

ArgoCD를 쓴다고 모든 환경을 무조건 자동 Sync로 둘 필요는 없음.

---

## 다음

[06. ApplicationSet](./06-ApplicationSet)

---

## 참고

- [Automated Sync Policy](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/)
- [Sync Options](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/)