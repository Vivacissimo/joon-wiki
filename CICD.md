---
title: CI/CD
description: CI/CD 학습 및 실습 문서 인덱스
published: true
date: 2026-08-27T07:01:00.000Z
tags: cicd,devops,jenkins,github-actions
editor: markdown
dateCreated: 2026-08-27T07:01:00.000Z
---

# CI/CD

CI/CD 관련 학습 내용과 실습 기록을 모아두는 인덱스 페이지다.

상세 문서는 `Study` 아래에서 관리한다.

## CI

### Jenkins

macOS 로컬 Jenkins에서 GitHub Repository를 연동하고, Push / Merge 시 CI가 자동 실행되도록 구성한 실습 기록.

- Pipeline script from SCM
- Jenkinsfile 자동 로드
- NodeJS Plugin / Node.js 22 Tool
- `npm install → build → test`
- GitHub Push / PR Merge 자동 CI
- GitHub CLI Webhook Forwarding

[Jenkins 문서 보기](/en/Study/CICD/Jenkins)

### GitHub Actions

추후 Jenkins와 동일한 기준으로 별도 실습 후 정리할 예정.

- Build / Test
- Docker Image Build
- ECR Push
- Branch / PR Trigger
- Jenkins와 비교

## CD / GitOps

CI 이후 배포는 ArgoCD 중심의 GitOps 구조로 정리할 예정.

```text
Developer
   ↓
GitHub
   ↓
CI
├─ Jenkins
└─ GitHub Actions
   ↓
Build / Test
   ↓
Docker Image Build
   ↓
ECR Push
   ↓
GitOps Repository
   ↓
ArgoCD
   ↓
EKS
```

## 문서 구조

```text
Study/
├── CICD.md
└── CICD/
    ├── Jenkins.md
    └── GitHub-Actions.md   # 추후 추가
```

현재 Jenkins 실습은 Build / Test 및 GitHub Push / Merge 자동 실행까지 완료했다.

다음 단계는 Jenkins에서 Docker Image를 빌드하고 AWS ECR에 Push하는 흐름이다.

[Study의 CI/CD 인덱스 보기](/en/Study/CICD)
