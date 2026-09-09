---
title: 09. Deployment and Rollback
description: ArgoCD 배포 흐름과 GitOps 기준 롤백 방법 정리
published: true
date: 2026-09-09T07:58:00.000Z
tags: argocd,cicd,gitops,kubernetes,rollback,rollouts
editor: markdown
dateCreated: 2026-09-09T07:58:00.000Z
---

# 09. Deployment / Rollback

## 기본 배포

ArgoCD 자체가 새로운 Image를 만드는 것은 아니다.

Git에 새 Image Tag가 들어오면 Kubernetes Deployment를 갱신한다.

```text
Git
image.tag: new-sha
      ↓
ArgoCD Sync
      ↓
Deployment Update
      ↓
새 ReplicaSet 생성
      ↓
새 Pod 생성
```

일반 Deployment라면 Kubernetes의 RollingUpdate 전략을 그대로 사용.

---

## Rolling Update

Kubernetes Deployment 기본 배포 방식.

기존 Pod를 한 번에 모두 내리지 않고 새 Pod로 순차 교체.

```text
v1 v1 v1
   ↓
v1 v1 v2
   ↓
v1 v2 v2
   ↓
v2 v2 v2
```

ArgoCD는 이 Deployment 설정을 Git에서 가져와 적용한다.

---

## ArgoCD와 Canary / Blue-Green

중요한 구분.

ArgoCD는 GitOps 동기화 도구.

Canary / Blue-Green 같은 세밀한 Progressive Delivery 기능은 보통 **Argo Rollouts**와 같이 사용.

```text
ArgoCD
= Git 상태를 Cluster에 동기화

Argo Rollouts
= Canary / Blue-Green 배포 진행 제어
```

구조.

```text
Git
 ↓
ArgoCD
 ↓
Rollout Resource
 ↓
Argo Rollouts Controller
 ↓
Canary / Blue-Green 진행
```

프로젝트 Phase 3의 카나리·블루그린 시연을 구현할 때는 `ArgoCD만으로 처리하는 기능`과 `Argo Rollouts가 필요한 기능`을 분리해서 보는 게 좋다.

---

## Rollback

배포 문제가 생겼을 때 이전 버전으로 돌리는 작업.

GitOps에서는 **Git이 최종 기준**.

그래서 일반적인 롤백은 이전 Git 상태로 되돌리는 방식으로 처리.

```text
잘못된 배포 Commit
      ↓
Git Revert
      ↓
GitOps Repository 이전 Tag 복구
      ↓
ArgoCD Sync
      ↓
이전 Image 배포
```

---

## Image Tag 롤백 예시

문제 버전.

```yaml
image:
  tag: f91ab23
```

이전 정상 버전.

```yaml
image:
  tag: a82f91c
```

Git에서 이전 값으로 되돌린 뒤 Commit.

```text
revert: rollback backend image to a82f91c
```

ArgoCD가 변경을 감지하고 다시 배포.

---

## ArgoCD History

Application 배포 이력 확인.

```bash
argocd app history backend-dev
```

Application 상태.

```bash
argocd app get backend-dev
```

ArgoCD CLI에는 Application History 기준 rollback 기능도 있다.

```bash
argocd app rollback backend-dev <history-id>
```

다만 Git 상태가 그대로라면 이후 Sync에서 다시 Git의 상태가 적용될 수 있다.

GitOps 운영에서는 **Git Revert까지 같이 처리**하는 흐름을 기준으로 잡는 편이 명확하다.

---

## 배포 실패 시 보는 상태

### Sync 실패

Manifest 자체를 적용하지 못한 상태.

예.

- 잘못된 YAML
- 존재하지 않는 Namespace
- 권한 문제
- CRD 미설치

### Sync 성공 + Health 실패

Manifest는 적용됐지만 Application이 정상 동작하지 않는 상태.

예.

- ImagePullBackOff
- CrashLoopBackOff
- Readiness Probe 실패
- Resource 부족

```text
Git 반영 성공
     ↓
Synced
     ↓
Pod 문제
     ↓
Degraded
```

---

## 배포 검증 흐름

```text
GitOps Commit
     ↓
ArgoCD Sync 확인
     ↓
Application Synced 확인
     ↓
Health Healthy 확인
     ↓
Pod Ready 확인
     ↓
Service / Ingress 확인
     ↓
실제 요청 확인
```

`Synced`만 보고 배포 성공으로 판단하지 않는다.

---

## 프로젝트에서 롤백 흐름

```text
Backend 새 버전 배포
       ↓
Health Check 실패
       ↓
GitOps backend values의 Tag 롤백
       ↓
Git Commit / Push
       ↓
ArgoCD Sync
       ↓
이전 Image로 복구
```

Jenkins를 다시 돌려서 이전 이미지를 새로 만들 필요는 없음.

ECR에 이전 Image Tag가 남아 있다면 해당 Tag로 GitOps 설정만 복구 가능.

그래서 Image Tag를 `latest` 하나로만 관리하지 않는 것이 중요.

---

## 다음

[10. Troubleshooting](./10-Troubleshooting)

---

## 참고

- [Argo CD Rollback Command](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_app_rollback/)
- [Argo Rollouts](https://argo-rollouts.readthedocs.io/en/stable/)
- [Kubernetes Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)