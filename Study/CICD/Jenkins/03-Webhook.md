---
title: Jenkins Webhook
description: Jenkins Trigger와 GitHub Webhook
published: true
tags: jenkins,github,webhook,cicd
editor: markdown
---

# Jenkins Webhook

## Trigger

Pipeline 실행 조건이다.

- 수동 실행
- Poll SCM
- Webhook
- Timer
- 다른 Job 완료 후 실행

<br>

## Poll SCM

Jenkins가 Git Repository 변경 여부를 주기적으로 확인한다.

```text
Jenkins
  ↓ 일정 주기
Git Repository 확인
  ↓ 변경 있음
Build
```

<br>

## GitHub Webhook

GitHub Push / Merge 이벤트 발생 시 Jenkins를 호출한다.

```text
Git Push / Merge
       ↓
     GitHub
       ↓ Webhook
     Jenkins
       ↓
    Pipeline
```

Jenkins 설정

```text
Configure
→ Build Triggers
→ GitHub hook trigger for GITScm polling
```

Webhook Endpoint 예시

```text
https://jenkins.example.com/github-webhook/
```

<br>

## Merge와 Push

PR을 대상 Branch에 Merge하면 대상 Branch 변경으로 Push 이벤트가 발생한다.

```text
Feature Branch
     ↓
Pull Request
     ↓
Merge
     ↓
Push Event
     ↓
Jenkins CI
```

<br>

## 로컬 Jenkins

GitHub 서버는 로컬 PC의 `localhost`에 직접 접근할 수 없다.

로컬 실습에서는 Tunnel, GitHub CLI Webhook Forwarding, Poll SCM 등을 사용할 수 있다.
