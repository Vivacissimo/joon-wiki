---
title: Jenkins
description: macOS 로컬 Jenkins 기반 CI 자동화 실습 기록
published: true
date: 2026-08-27T07:01:00.000Z
tags: jenkins,cicd,devops,github,webhook,nodejs
editor: markdown
dateCreated: 2026-08-27T07:01:00.000Z
---

# Jenkins

Jenkins의 기본 개념과 macOS 로컬 환경에서 GitHub Repository를 연동해 CI를 자동화한 실습 기록이다.

GitHub Actions는 별도 문서로 분리해서 정리할 예정이며, 이 문서는 Jenkins에 집중한다.

## 1. Jenkins란

Jenkins는 오픈소스 자동화 서버로, 코드 변경을 기준으로 Build, Test, Image Build, Registry Push 같은 CI 작업을 자동화할 수 있다.

기본 흐름은 다음과 같다.

```text
개발자
  ↓ push / merge
GitHub
  ↓
Jenkins
  ├─ Checkout
  ├─ Install
  ├─ Build
  └─ Test
  ↓
SUCCESS / FAILURE
```

현재 실습에서는 **CI까지만 Jenkins가 담당**하고, 향후 배포는 ArgoCD를 사용하는 GitOps 구조로 분리할 계획이다.

```text
Jenkins = CI
ArgoCD  = CD
```

## 2. 실습 환경

- OS: macOS
- Jenkins: 로컬 설치
- Jenkins URL: `http://localhost:8080`
- Test Repository: `Vivacissimo/jenkins-test`
- Branch: `main`
- Pipeline: `Pipeline script from SCM`
- Jenkinsfile: Repository Root
- Runtime: Node.js 22

## 3. Repository 구조

```text
jenkins-test/
├── Jenkinsfile
├── app.js
└── package.json
```

`package.json`의 Script는 아래 흐름으로 사용했다.

```json
{
  "scripts": {
    "build": "echo build success",
    "test": "node app.js"
  }
}
```

## 4. Jenkins Job 설정

Jenkins에서 Pipeline Job을 생성하고 아래처럼 설정했다.

```text
New Item
→ Pipeline
```

Pipeline 설정:

```text
Definition
→ Pipeline script from SCM

SCM
→ Git

Repository URL
→ https://github.com/Vivacissimo/jenkins-test.git

Branch Specifier
→ */main

Script Path
→ Jenkinsfile
```

이 설정을 사용하면 Jenkins가 Repository의 최신 `Jenkinsfile`을 가져와 Pipeline을 실행한다.

## 5. Jenkinsfile

현재 실습에서 사용한 Jenkinsfile은 다음과 같다.

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

Pipeline 흐름:

```text
Git Checkout
    ↓
npm install
    ↓
npm run build
    ↓
npm test
    ↓
CI SUCCESS / FAILURE
```

## 6. Node.js Tool 설정

처음 실행했을 때 Jenkins에서 아래 오류가 발생했다.

```text
npm: command not found
script returned exit code 127
```

macOS 터미널에서 사용하는 PATH와 Jenkins 서비스가 사용하는 실행 환경이 달라 Jenkins에서 `npm`을 찾지 못한 것이 원인이었다.

### 해결

Jenkins에 **NodeJS Plugin**을 설치하고 Node.js Tool을 등록했다.

```text
Manage Jenkins
→ Plugins
→ NodeJS 설치
```

이후:

```text
Manage Jenkins
→ Tools
→ NodeJS installations
```

설정:

```text
Name: node22
Install automatically: 활성화
Node.js: 22.x
```

그리고 Jenkinsfile에 다음을 추가했다.

```groovy
tools {
    nodejs 'node22'
}
```

이후 Jenkins가 필요한 Node.js 환경을 자동으로 준비하면서 `npm install`, `npm run build`, `npm test`가 정상 실행됐다.

## 7. Checkout 중복 제거

`Pipeline script from SCM`을 사용하는 Declarative Pipeline에서는 Jenkins가 Pipeline 시작 시 자동으로 SCM Checkout을 수행한다.

로그에서 아래 Stage가 자동으로 생성된다.

```text
Declarative: Checkout SCM
```

따라서 Jenkinsfile에 아래 내용을 별도로 넣으면 Checkout이 두 번 발생한다.

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

현재는 별도 Checkout Stage를 제거하고 Jenkins 기본 Checkout만 사용하고 있다.

## 8. GitHub Push / Merge 자동 CI

초기에는 GitHub에 Push한 뒤 Jenkins에서 직접 `Build Now`를 눌러야 했다.

목표는 다음처럼 변경했다.

```text
git push / PR merge
        ↓
GitHub
        ↓
Webhook
        ↓
Jenkins
        ↓
CI 자동 실행
```

### Jenkins Trigger 설정

Jenkins Job에서:

```text
Configure
→ Build Triggers
→ GitHub hook trigger for GITScm polling
```

을 활성화했다.

## 9. 로컬 Jenkins Webhook 문제

Jenkins가 아래 주소에서 실행 중이기 때문에 GitHub 서버가 직접 접근할 수 없다.

```text
http://localhost:8080
```

GitHub 서버 입장에서 `localhost`는 사용자 Mac이 아니기 때문이다.

```text
GitHub
   │
   └──X──→ localhost:8080
```

로컬 실습에서는 GitHub CLI의 Webhook Forwarding을 사용했다.

## 10. GitHub CLI Webhook Forwarding

GitHub CLI 설치 및 인증 후 webhook extension을 설치한다.

```bash
gh extension install cli/gh-webhook
```

Webhook Forwarding 실행:

```bash
gh webhook forward \
  --repo=Vivacissimo/jenkins-test \
  --events=push \
  --url=http://localhost:8080/github-webhook/
```

이 명령을 실행 중인 동안 GitHub의 Push 이벤트가 로컬 Jenkins로 전달된다.

```text
GitHub
   ↓ push event
GitHub CLI
   ↓ forwarding
localhost:8080/github-webhook/
   ↓
Jenkins
```

## 11. Push 시 자동 실행

`main`에 직접 Push하면 다음 흐름으로 자동 CI가 실행되는 것을 확인했다.

```text
git push origin main
        ↓
GitHub Push Event
        ↓
Webhook
        ↓
Jenkins
        ↓
Jenkinsfile 실행
        ↓
Install → Build → Test
```

## 12. PR Merge 시 자동 실행

Feature Branch에서 PR을 생성한 뒤 `main`으로 Merge하는 경우에도 `main`이 변경되므로 Push 이벤트가 발생한다.

```text
feature branch
       ↓
Pull Request
       ↓
Merge to main
       ↓
main 변경
       ↓
Push Event
       ↓
Webhook
       ↓
Jenkins CI
```

즉 현재 구성에서는 아래 두 경우 모두 자동 CI가 실행된다.

- `main` 직접 Push
- PR을 `main`으로 Merge

## 13. 성공 로그

최종적으로 아래 단계가 모두 성공했다.

```text
GitHub 최신 Commit Checkout
        ↓
Node.js 22 Tool 준비
        ↓
npm install
        ↓
npm run build
        ↓
npm test
        ↓
CI SUCCESS
        ↓
Finished: SUCCESS
```

## 14. 현재 완료 범위

- [x] macOS Jenkins 설치
- [x] Jenkins Pipeline Job 생성
- [x] GitHub Repository 연동
- [x] Pipeline script from SCM 설정
- [x] Jenkinsfile 자동 로드
- [x] 최신 `main` Commit Checkout
- [x] NodeJS Plugin 설정
- [x] Node.js 22 Tool 설정
- [x] `npm install`
- [x] Build
- [x] Test
- [x] Pipeline 성공/실패 처리
- [x] 중복 Checkout 제거
- [x] GitHub Push 시 CI 자동 실행
- [x] PR Merge 후 CI 자동 실행

## 15. 다음 실습

현재는 코드 Build/Test까지 완료한 상태다.

다음 단계는 Jenkins에서 배포 가능한 Container Image를 만드는 흐름이다.

```text
GitHub
  ↓
Jenkins
  ├─ Checkout
  ├─ Install
  ├─ Build
  ├─ Test
  ├─ Docker Build
  └─ ECR Push
```

추가할 내용:

1. Dockerfile 작성
2. Jenkins에서 Docker Image Build
3. Image Tag에 `BUILD_NUMBER` 또는 Git Commit SHA 적용
4. AWS ECR 인증
5. ECR Push
6. GitOps Repository의 Image Tag 변경
7. ArgoCD가 변경을 감지해 EKS에 배포

## 16. 최종적으로 가져갈 CI/CD 역할 분리

향후 실제 프로젝트에서는 Jenkins가 Kubernetes에 직접 배포하기보다 CI와 CD를 분리한다.

```text
Developer
   ↓
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
EKS
```

### Jenkins

- Checkout
- Build
- Test
- Container Image Build
- Registry Push
- GitOps Repository Image Tag 갱신

### ArgoCD

- GitOps Repository 변경 감지
- Kubernetes Manifest 동기화
- EKS 배포
- Application 상태 확인

이 구조를 기준으로 Jenkins와 GitHub Actions를 각각 실습한 뒤 CI 도구의 차이를 비교할 예정이다.
