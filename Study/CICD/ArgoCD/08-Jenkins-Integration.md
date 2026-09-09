---
title: 08. Jenkins Integration
description: Jenkins CI와 ArgoCD GitOps CD 연결 흐름 정리
published: true
date: 2026-09-09T07:57:00.000Z
tags: argocd,jenkins,cicd,gitops,ecr
editor: markdown
dateCreated: 2026-09-09T07:57:00.000Z
---

# 08. Jenkins Integration

## 역할 분리

Jenkins와 ArgoCD를 같이 사용할 때 가장 먼저 볼 부분.

```text
Jenkins
= CI + 배포 정보 변경

ArgoCD
= Kubernetes CD
```

전체 흐름.

```text
Application Repository
        ↓
      Jenkins
        ↓
 Build / Test
        ↓
Docker Image Build
        ↓
     ECR Push
        ↓
GitOps Repository Image Tag 변경
        ↓
      ArgoCD
        ↓
       EKS
```

---

## Jenkins가 담당하는 구간

예.

```text
1. Source Checkout
2. Build
3. Test
4. Docker Image Build
5. ECR Login
6. ECR Push
7. GitOps Repository Clone
8. values.yaml Image Tag 수정
9. Commit / Push
```

여기까지 끝나면 Jenkins의 배포 관련 역할도 끝.

그 이후는 ArgoCD가 처리.

---

## ArgoCD가 담당하는 구간

```text
GitOps Repository 변경 감지
        ↓
Desired State 변경 확인
        ↓
OutOfSync
        ↓
Sync
        ↓
Deployment Update
        ↓
새 Pod 생성
        ↓
Healthy 확인
```

---

## Jenkins에서 직접 kubectl apply를 하지 않는 이유

GitOps를 적용하면서 Jenkins도 Cluster를 직접 수정하면 배포 주체가 두 개가 된다.

```text
Jenkins ─────→ Kubernetes

ArgoCD  ─────→ Kubernetes
```

이 경우 실제 상태가 왜 바뀌었는지 추적하기 어려워진다.

GitOps 기준 흐름은 아래처럼 한 방향으로 잡는다.

```text
Jenkins
   ↓
Git
   ↓
ArgoCD
   ↓
Kubernetes
```

즉 Jenkins에서는 가능하면 아래 작업을 하지 않는다.

```bash
kubectl apply -f ...
helm upgrade ...
```

ArgoCD가 관리하는 Application 배포라면 Git을 수정.

---

## Image Tag

Image Tag는 Git Commit SHA 기반으로 두면 추적하기 쉽다.

예.

```text
backend:a82f91c
```

ECR.

```text
123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/backend:a82f91c
```

GitOps values.

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/backend
  tag: a82f91c
```

Application Code Commit과 배포 Image를 연결할 수 있음.

---

## GitOps Tag 변경 예시

기존.

```yaml
image:
  tag: 91bd210
```

Jenkins Build 완료.

```text
a82f91c
```

GitOps Repository 수정.

```yaml
image:
  tag: a82f91c
```

Commit.

```text
chore: update backend dev image to a82f91c
```

ArgoCD가 이 Commit을 감지.

---

## Jenkins Pipeline 안에서 보면

개념 예시.

```groovy
stage('Build') {
    steps {
        sh './gradlew build'
    }
}

stage('Docker Build') {
    steps {
        sh 'docker build -t backend:${GIT_COMMIT} .'
    }
}

stage('ECR Push') {
    steps {
        sh 'docker push ${ECR_URL}/backend:${GIT_COMMIT}'
    }
}

stage('Update GitOps') {
    steps {
        sh '''
          git clone <gitops-repository>
          cd gitops
          # values.yaml의 image.tag 변경
          git add .
          git commit -m "chore: update backend image"
          git push
        '''
    }
}
```

실제 Credential 처리와 Tag 수정 도구는 Pipeline 구성에 맞춰 따로 잡는다.

---

## Branch와 환경 연결

예.

```text
develop Merge
      ↓
Jenkins Dev Pipeline
      ↓
backend:<commit-sha> ECR Push
      ↓
environments/dev/backend/values.yaml 수정
      ↓
ArgoCD
      ↓
Dev 배포
```

```text
main Merge
    ↓
Jenkins Demo / Prod Pipeline
    ↓
Image Push
    ↓
environments/prod/backend/values.yaml 수정
    ↓
ArgoCD
    ↓
Demo / Prod 배포
```

Branch와 EKS Environment는 실제 운영 정책에 맞춰 연결.

---

## FE / BE / AI가 있을 때

각 Application Repository에서 자기 Image만 만든다.

```text
Frontend Repo
   ↓ Jenkins
frontend:<sha>

Backend Repo
   ↓ Jenkins
backend:<sha>

AI Repo
   ↓ Jenkins
ai:<sha>
```

GitOps Repository.

```text
environments/dev/
├─ frontend/values.yaml
├─ backend/values.yaml
└─ ai/values.yaml
```

Jenkins는 해당 서비스의 `image.tag`만 변경.

ArgoCD는 전체 GitOps Repository를 기준으로 각 Application을 관리.

---

## 장애가 났을 때 확인 순서

배포가 안 됐다면 어디에서 끊겼는지 순서대로 확인.

```text
1. Jenkins Build 성공?
2. ECR Image 존재?
3. GitOps values Tag 변경?
4. Git Push 성공?
5. ArgoCD OutOfSync 감지?
6. Sync 성공?
7. Pod 정상 생성?
8. Health Check 통과?
```

CI 문제와 CD 문제를 섞어서 보지 않는 게 중요.

---

## 정리

```text
Jenkins는 Cluster에 배포하지 않는다.

Jenkins
→ Image 생성
→ ECR Push
→ GitOps Repository 수정

ArgoCD
→ Git 감시
→ Kubernetes 배포
```

---

## 다음

[09. Deployment / Rollback](./09-Deployment-Rollback)

---

## 참고

- [Argo CD CI Automation](https://argo-cd.readthedocs.io/en/stable/user-guide/ci_automation/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)