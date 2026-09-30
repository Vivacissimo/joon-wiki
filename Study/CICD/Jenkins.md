---
title: Jenkins
description: Jenkins 개념과 CI Pipeline 구성 정리
published: true
date: 2026-08-27T07:01:00.000Z
tags: jenkins,cicd,devops,github,webhook
editor: markdown
dateCreated: 2026-08-27T07:01:00.000Z
---

# Jenkins

## 개념

- 오픈소스 자동화 서버
- CI/CD Pipeline 구성 가능
- Build, Test, Image Build, 배포 작업 자동화
- 다양한 Plugin 지원
- 자체 서버에 설치해서 운영

```text
Code Push
   ↓
Jenkins
   ├─ Checkout
   ├─ Build
   ├─ Test
   └─ Image Build
```

## Jenkins 구성

### Controller

- Jenkins 관리 영역
- Job / Pipeline 관리
- Plugin 관리
- Build 스케줄링
- Agent에 작업 전달

### Agent

- 실제 Build 작업 실행
- Build / Test / Docker Build 수행
- 별도 서버, VM, Container, Kubernetes Pod 사용 가능

```text
Jenkins Controller
        ↓
     Agent
   ├─ Build
   ├─ Test
   └─ Docker Build
```

작은 환경에서는 Controller 자체에서 Build 가능.

운영 환경에서는 Controller와 Agent 분리 권장.

---

## Job

Jenkins에서 실행할 작업 단위.

대표 유형

- Freestyle Project
- Pipeline
- Multibranch Pipeline

### Pipeline

Pipeline 전체 흐름을 코드로 관리.

```text
Checkout
   ↓
Build
   ↓
Test
   ↓
Docker Build
   ↓
Registry Push
```

---

## Jenkinsfile

Jenkins Pipeline을 코드로 정의하는 파일.

Repository Root에 주로 배치.

```text
project/
├── Jenkinsfile
├── src/
└── package.json
```

장점

- Pipeline 설정 Git 관리
- 코드 변경 이력 확인 가능
- 프로젝트별 Pipeline 관리 가능
- Jenkins UI 설정 의존도 감소

---

## Declarative Pipeline

Jenkinsfile 작성 방식 중 하나.

기본 구조

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
```

### 주요 구성

`pipeline`
- Pipeline 전체 영역

`agent`
- Pipeline 실행 위치

`stages`
- Pipeline 단계 묶음

`stage`
- Build, Test 같은 개별 단계

`steps`
- 실제 명령 실행

`post`
- 성공 / 실패 후 처리

---

## Pipeline script from SCM

Jenkinsfile을 Jenkins UI에 직접 작성하지 않고 Git Repository에서 읽는 방식.

```text
GitHub Repository
       ↓
   Jenkinsfile
       ↓
     Jenkins
```

설정 위치

```text
Pipeline
→ Definition
→ Pipeline script from SCM
```

필요 항목

- SCM: Git
- Repository URL
- Branch
- Script Path: `Jenkinsfile`

장점

- Jenkinsfile과 Application Code 같이 관리
- Push된 최신 Pipeline 코드 사용
- Pipeline 변경 이력 Git에서 확인

---

## SCM Checkout

`Pipeline script from SCM` 사용 시 Jenkins가 기본 Checkout 수행.

로그 예시

```text
Declarative: Checkout SCM
```

따라서 아래 Stage를 별도로 추가하면 중복 Checkout 가능.

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

특별한 Checkout 처리가 없다면 기본 Checkout만 사용.

---

## Plugin

Jenkins 기능 확장 단위.

예시

- Git Plugin
- GitHub Plugin
- NodeJS Plugin
- Docker Pipeline Plugin
- Kubernetes Plugin
- Credentials Plugin

필요한 기능만 설치하는 방식이 관리에 유리.

Plugin 버전 / 보안 업데이트 관리 필요.

---

## Tool 관리

Build에 필요한 Runtime / Tool을 Jenkins에서 관리 가능.

예시

- JDK
- Maven
- Gradle
- Node.js

NodeJS Plugin 예시

```groovy
tools {
    nodejs 'node22'
}
```

Jenkins에 등록된 `node22` Tool 사용.

장점

- Agent마다 동일한 Tool 이름 사용
- 로컬 PATH 의존성 감소
- Pipeline 실행 환경 통일

---

## Trigger

Pipeline 실행 조건.

대표 방식

- 수동 실행
- Poll SCM
- Webhook
- Timer
- 다른 Job 완료 후 실행

### 수동 실행

```text
Build Now
```

테스트할 때 간단한 방식.

### Poll SCM

Jenkins가 Git Repository 변경 여부를 주기적으로 확인.

```text
Jenkins
  ↓ 일정 주기
Git Repository 확인
  ↓ 변경 있음
Build
```

불필요한 Polling 발생 가능.

### Webhook

GitHub 이벤트를 Jenkins로 바로 전달.

```text
Git Push / Merge
       ↓
     GitHub
       ↓ Webhook
     Jenkins
       ↓
    Pipeline
```

Push 기반 CI에서 주로 사용.

---

## GitHub Webhook

GitHub Push / Merge 이벤트 발생 시 Jenkins 호출.

Jenkins Trigger

```text
Configure
→ Build Triggers
→ GitHub hook trigger for GITScm polling
```

Webhook Endpoint

```text
https://jenkins.example.com/github-webhook/
```

### Merge와 Push

PR을 `main`에 Merge하면 `main` 변경 발생.

결과적으로 Push 이벤트 발생.

```text
Feature Branch
     ↓
Pull Request
     ↓
Merge to main
     ↓
Push Event
     ↓
Jenkins CI
```

---

## 로컬 Jenkins와 Webhook

로컬 Jenkins 예시

```text
http://localhost:8080
```

GitHub 서버에서 로컬 PC의 `localhost` 접근 불가.

```text
GitHub ──X──> localhost:8080
```

로컬 실습용 대안

- GitHub CLI Webhook Forwarding
- ngrok / Cloudflare Tunnel 같은 Tunnel
- Poll SCM

### GitHub CLI Forwarding 예시

```bash
gh extension install cli/gh-webhook
```

```bash
gh webhook forward \
  --repo=Vivacissimo/jenkins-test \
  --events=push \
  --url=http://localhost:8080/github-webhook/
```

흐름

```text
GitHub
   ↓
GitHub CLI
   ↓
localhost:8080/github-webhook/
   ↓
Jenkins
```

---

## Credentials

외부 서비스 인증정보 관리 기능.

예시

- GitHub Token
- AWS Access Key
- Docker Registry 계정
- SSH Key

Jenkinsfile에 Secret 직접 작성 금지.

```text
Jenkins Credentials
       ↓
    Pipeline
       ↓
GitHub / ECR / AWS
```

---

## CI에서 Jenkins 역할

일반적인 CI 범위

```text
Git Push
   ↓
Jenkins
   ├─ Checkout
   ├─ Dependency Install
   ├─ Build
   ├─ Test
   ├─ Docker Build
   └─ Registry Push
```

Jenkins에서 Kubernetes에 직접 배포하는 방식도 가능.

GitOps 환경에서는 CI / CD 분리 가능.

```text
GitHub
   ↓
Jenkins
   ├─ Build
   ├─ Test
   ├─ Docker Build
   └─ ECR Push
   ↓
GitOps Repository
   ↓
ArgoCD
   ↓
Kubernetes
```

### Jenkins

- CI
- Build
- Test
- Container Image Build
- Registry Push
- GitOps Repository Image Tag 변경

### ArgoCD

- CD
- Git 변경 감지
- Kubernetes Manifest Sync
- 배포 상태 확인

---

## 실습에서 사용한 Jenkinsfile

```groovy
pipeline {
    agent any

    tools {
        nodejs 'node22'
    }

    stages {
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }

    post {
        success {
            echo 'CI SUCCESS'
        }

        failure {
            echo 'CI FAILED'
        }
    }
}
```

현재 확인한 흐름

```text
GitHub Push / Merge
        ↓
      Webhook
        ↓
      Jenkins
        ↓
      Build
        ↓
       Test
        ↓
    CI SUCCESS
```

---

## 다음 정리

- Docker Build
- Image Tag 전략
- AWS ECR Push
- Jenkins Credentials
- Kubernetes Agent
- Jenkins 동적 Agent
- Jenkins vs GitHub Actions


---

## GitHub App 기반 GitOps PR Auto Merge

GitOps Repository의 이미지 태그 변경 PR을 Jenkins에서 자동 생성하고 자동 Merge하는 구성.

### 목적

일반 사용자 PR의 Review 정책은 유지하면서 Jenkins가 생성한 GitOps PR만 자동 Merge한다.

사용자 PAT를 사용하면 PR 생성과 Auto Merge 요청의 주체가 개인 계정이 된다.

```text
Jenkins
  ↓
User PAT
  ↓
GitOps PR 생성
  ↓
Ruleset Review 조건 적용
  ↓
승인 대기
```

GitHub App을 Jenkins 전용 인증 주체로 사용하고 Ruleset Bypass 대상에 등록한다.

```text
Jenkins
  ↓
GitHub App Credential
  ↓
Installation Access Token
  ↓
GitOps Branch Push
  ↓
PR 생성
  ↓
Auto Merge 활성화
  ↓
Ruleset Bypass
  ↓
Merge
  ↓
ArgoCD Sync
```

<br>

### GitHub App

프로젝트에서는 Jenkins GitOps 자동화 전용 GitHub App을 사용한다.

```text
moongcheap-jenkins-ci
```

주요 Repository 권한

- Metadata: Read
- Checks: Read / Write
- Contents: Read / Write
- Commit statuses: Read / Write
- Pull requests: Read / Write

GitHub App은 Organization에 설치하고 필요한 Repository에 접근 권한을 부여한다.

<br>

### Ruleset Bypass

`develop` Branch 보호 Ruleset에 GitHub App을 Bypass 대상으로 등록한다.

```text
Ruleset
└─ protect-develop
   ├─ Target: develop
   └─ Bypass
      └─ moongcheap-jenkins-ci
         └─ Always allow
```

Bypass는 자동 Merge 기능 자체가 아니다.

GitHub App이 Ruleset의 Review 조건 등을 통과할 수 있게 하는 설정이다.

따라서 Jenkinsfile에서 PR 생성 후 Auto Merge 요청까지 실행해야 한다.

<br>

### Jenkins Credential

기존에는 사용자 PAT Credential을 사용했다.

```text
gitops-repo-push
└─ User PAT
```

이 경우 GitHub에서 작업 주체를 사용자로 인식하기 때문에 GitHub App에 설정한 Bypass가 적용되지 않는다.

GitHub App의 App ID와 Private Key를 Jenkins Credential에 등록한다.

```text
Kind: GitHub App
ID: moongcheap-jenkins-ci
```

Private Key는 GitHub App에서 생성한 `.pem` 파일의 전체 내용을 사용한다.

Private Key는 Repository나 Jenkinsfile에 직접 저장하지 않는다.

<br>

### Jenkinsfile Credential

GitHub App Credential은 Pipeline에서 `usernamePassword`로 가져온다.

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'moongcheap-jenkins-ci',
        usernameVariable: 'GIT_USER',
        passwordVariable: 'GIT_TOKEN'
    )
]) {
    // GitOps 작업
}
```

Jenkins가 GitHub App Credential을 이용해 Installation Access Token을 발급하고 `GIT_TOKEN`으로 전달한다.

이 Token으로 다음 작업을 처리한다.

- GitOps Repository Clone
- 임시 Branch Push
- Pull Request 생성
- Auto Merge 활성화
- PR Merge 상태 확인
- Merge 후 임시 Branch 삭제

<br>

### PR 생성

Jenkins에서 이미지 태그 변경 후 임시 Branch를 Push한다.

```text
ci/update-{service}-{image-tag}-{build-number}
```

이 Branch를 기준으로 GitOps PR을 생성한다.

```text
ci/update-backend-develop-xxxxxxx-10
        ↓
Pull Request
        ↓
develop
```

API 인증은 GitHub App Installation Token을 사용한다.

```bash
-H "Authorization: Bearer ${GIT_TOKEN}"
```

<br>

### Auto Merge

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

이 설정으로 PR이 Merge 가능한 상태가 되면 자동으로 Squash Merge된다.

현재 구성에서는 `moongcheap-jenkins-ci`가 Ruleset Bypass 대상이므로 Jenkins가 생성한 GitOps PR은 별도 사용자 Review 없이 처리할 수 있다.

<br>

### Merge 완료 확인

Jenkins는 Auto Merge 활성화 후 PR 상태를 반복 확인한다.

현재 구성

```text
MAX_RETRIES=30
RETRY_INTERVAL=10초
```

최대 약 5분 동안 Merge 완료 여부를 확인한다.

Merge가 완료되면 임시 Branch를 삭제한다.

```text
PR Merged
   ↓
Temporary Branch Delete
   ↓
GitOps Repository develop 변경
   ↓
ArgoCD 변경 감지
   ↓
Kubernetes Sync
```

<br>

### 전체 흐름

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

<br>

### 확인 포인트

자동 Merge가 되지 않을 경우 아래 순서로 확인한다.

1. Jenkins가 사용하는 Credential이 사용자 PAT인지 GitHub App인지 확인
2. GitHub App이 대상 Repository에 설치되어 있는지 확인
3. GitHub App의 Contents / Pull requests 권한 확인
4. 대상 Branch Ruleset의 Bypass list 확인
5. Jenkins 로그에서 PR 생성 성공 여부 확인
6. Auto Merge GraphQL 요청 성공 여부 확인
7. PR의 작업 주체가 GitHub App으로 기록되는지 확인

핵심은 **GitHub App을 설치하는 것만으로 자동 Merge되는 것이 아니라 Jenkins가 실제 GitHub App Credential로 API를 호출해야 한다는 점**이다.
