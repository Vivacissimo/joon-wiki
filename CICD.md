---
title: CI/CD
description: CI/CD 도구와 파이프라인 실습 정리
published: true
date: 2026-08-27T07:01:00.000Z
tags: cicd,devops,jenkins,github-actions
editor: markdown
dateCreated: 2026-08-27T07:01:00.000Z
---

# CI/CD

CI/CD 도구별 개념, 설정 방법, 실습 기록을 정리하는 공간이다.

## CI

- [Jenkins](./CICD/Jenkins)
- GitHub Actions — 추후 추가

## CD / GitOps

추후 ArgoCD, Kubernetes 배포 자동화 내용을 정리한다.

## 정리 방향

도구 자체 설명만 적기보다 아래 흐름을 기준으로 실습 결과를 기록한다.

```text
Source Repository
        ↓
CI
├─ Checkout
├─ Install
├─ Build
├─ Test
├─ Image Build
└─ Registry Push
        ↓
GitOps Repository
        ↓
CD
        ↓
Kubernetes
```

Jenkins와 GitHub Actions는 각각 별도 문서로 관리하고, 실제 프로젝트에서 어떤 기준으로 선택하는지도 추후 비교 정리한다.
