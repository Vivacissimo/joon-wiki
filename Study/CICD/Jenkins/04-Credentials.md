---
title: Jenkins Credentials
description: Jenkins Secret과 GitHub App Credential 관리
published: true
tags: jenkins,credentials,github-app,security
editor: markdown
---

# Jenkins Credentials

## 개념

외부 서비스 인증정보를 Jenkins에서 관리하는 기능이다.

예시

- GitHub Token
- GitHub App
- AWS Access Key
- Docker Registry 계정
- SSH Key

Secret은 Jenkinsfile에 직접 작성하지 않는다.

```text
Jenkins Credentials
       ↓
    Pipeline
       ↓
GitHub / ECR / AWS
```

<br>

## GitHub App Credential

MoongCheap GitOps 자동화에서는 Jenkins 전용 GitHub App을 사용한다.

```text
moongcheap-jenkins-ci
```

GitHub App의 App ID와 Private Key를 Jenkins Credential에 등록한다.

```text
Kind: GitHub App
ID: moongcheap-jenkins-ci
```

Private Key는 GitHub App에서 생성한 `.pem` 파일의 전체 내용을 사용한다.

Repository나 Jenkinsfile에는 Private Key를 저장하지 않는다.

<br>

## Jenkinsfile에서 사용

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'moongcheap-jenkins-ci',
        usernameVariable: 'GIT_USER',
        passwordVariable: 'GIT_TOKEN'
    )
]) {
    // GitHub API / Git 작업
}
```

Jenkins가 GitHub App Credential을 이용해 Installation Access Token을 발급한다.

```text
GitHub App
   ↓
App ID + Private Key
   ↓
Jenkins Credential
   ↓
Installation Access Token
   ↓
GIT_TOKEN
```

이 Token으로 GitOps Repository Push, PR 생성, Auto Merge 요청 등을 처리한다.
