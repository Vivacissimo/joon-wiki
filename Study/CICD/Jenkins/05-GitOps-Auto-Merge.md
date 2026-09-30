---
title: Jenkins GitOps Auto Merge
description: GitHub App 기반 GitOps PR 자동 생성 및 자동 Merge
published: true
tags: jenkins,gitops,github-app,auto-merge,argocd
editor: markdown
---

# GitOps PR Auto Merge

## 목적

일반 사용자 PR의 Review 정책은 유지하면서 Jenkins가 생성한 GitOps PR은 자동으로 Merge한다.

기존 사용자 PAT를 사용하면 GitHub에서 작업 주체를 개인 계정으로 인식한다.

```text
Jenkins
  ↓
User PAT
  ↓
GitOps PR
  ↓
Ruleset Review 조건
  ↓
승인 대기
```

Jenkins 전용 GitHub App을 사용하고 Branch Ruleset의 Bypass 대상으로 등록한다.

<br>

## GitHub App

```text
moongcheap-jenkins-ci
```

주요 Repository 권한

- Metadata: Read
- Checks: Read / Write
- Contents: Read / Write
- Commit statuses: Read / Write
- Pull requests: Read / Write

GitHub App은 Organization에 설치하고 대상 Repository 접근 권한을 부여한다.

<br>

## Ruleset Bypass

`develop` Branch 보호 Ruleset 구성이다.

```text
Ruleset
└─ protect-develop
   ├─ Target: develop
   └─ Bypass
      └─ moongcheap-jenkins-ci
         └─ Always allow
```

Bypass는 Auto Merge를 실행하는 기능이 아니다.

GitHub App이 Ruleset의 Review 조건 등을 우회할 수 있게 한다.

PR 생성 후 Auto Merge 활성화는 Jenkins Pipeline에서 별도로 수행한다.

<br>

## PR 생성

Jenkins에서 이미지 태그를 변경하고 임시 Branch를 Push한다.

```text
ci/update-{service}-{image-tag}-{build-number}
```

예시

```text
ci/update-backend-develop-xxxxxxx-10
        ↓
Pull Request
        ↓
develop
```

API 인증에는 GitHub App Installation Token을 사용한다.

```bash
-H "Authorization: Bearer ${GIT_TOKEN}"
```

<br>

## Auto Merge 활성화

PR 생성 후 GraphQL의 `enablePullRequestAutoMerge`를 호출한다.

```graphql
mutation {
  enablePullRequestAutoMerge(
    input: {
      pullRequestId: "PR_NODE_ID"
      mergeMethod: SQUASH
    }
  ) {
    pullRequest {
      number
      autoMergeRequest {
        enabledAt
      }
    }
  }
}
```

PR이 Merge 가능한 상태가 되면 Squash Merge된다.

`moongcheap-jenkins-ci`는 Ruleset Bypass 대상이므로 Jenkins가 생성한 GitOps PR은 별도 사용자 Review 없이 처리할 수 있다.

<br>

## Merge 확인

Jenkins는 Auto Merge 활성화 후 PR 상태를 반복 확인한다.

```text
MAX_RETRIES=30
RETRY_INTERVAL=10초
```

최대 약 5분 동안 Merge 완료 여부를 확인한다.

Merge 완료 후 임시 Branch를 삭제한다.

<br>

## 전체 흐름

```text
Application develop Merge
        ↓
      Jenkins
        ↓
   Build / Test
        ↓
 Container Build
        ↓
     ECR Push
        ↓
GitOps Image Tag 변경
        ↓
Temporary Branch Push
        ↓
     PR 생성
        ↓
Auto Merge 활성화
        ↓
moongcheap-jenkins-ci
Ruleset Bypass
        ↓
   Squash Merge
        ↓
Temporary Branch 삭제
        ↓
     ArgoCD
        ↓
   Kubernetes
```
