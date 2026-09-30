---
title: Jenkins
description: Jenkins 학습 문서
published: true
tags: jenkins,cicd,devops
editor: markdown
---

# Jenkins

Jenkins 개념부터 Pipeline, GitHub 연동, GitOps 자동화까지 정리한다.

## 학습 순서

1. [Overview](./Jenkins/01-Overview.md)
2. [Pipeline](./Jenkins/02-Pipeline.md)
3. [Webhook](./Jenkins/03-Webhook.md)
4. [Credentials](./Jenkins/04-Credentials.md)
5. [GitOps PR Auto Merge](./Jenkins/05-GitOps-Auto-Merge.md)
6. [Troubleshooting](./Jenkins/06-Troubleshooting.md)

<br>

## 전체 흐름

```text
GitHub
  ↓
Webhook
  ↓
Jenkins
  ├─ Build
  ├─ Test
  ├─ Container Build
  └─ ECR Push
  ↓
GitOps Repository
  ↓
PR / Auto Merge
  ↓
ArgoCD
  ↓
Kubernetes
```

<br>

## 문서 구성

```text
Study/CICD/
├── Jenkins.md
└── Jenkins/
    ├── 01-Overview.md
    ├── 02-Pipeline.md
    ├── 03-Webhook.md
    ├── 04-Credentials.md
    ├── 05-GitOps-Auto-Merge.md
    └── 06-Troubleshooting.md
```
