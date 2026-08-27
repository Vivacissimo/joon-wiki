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
