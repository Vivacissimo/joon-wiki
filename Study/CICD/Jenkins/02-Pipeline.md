---
title: Jenkins Pipeline
description: Jenkinsfile과 Pipeline 구성
published: true
tags: jenkins,pipeline,jenkinsfile,cicd
editor: markdown
---

# Jenkins Pipeline

## Jenkinsfile

Jenkins Pipeline을 코드로 정의하는 파일이다.

Repository Root에 주로 배치한다.

```text
project/
├── Jenkinsfile
├── src/
└── package.json
```

- Pipeline 설정을 Git으로 관리
- 변경 이력 확인
- 프로젝트별 Pipeline 관리
- Jenkins UI 설정 의존도 감소

<br>

## Declarative Pipeline

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

주요 구성

- `pipeline`: 전체 Pipeline
- `agent`: 실행 위치
- `stages`: 단계 묶음
- `stage`: 개별 단계
- `steps`: 실제 명령
- `post`: 성공/실패 후 처리

<br>

## Pipeline script from SCM

Jenkinsfile을 Jenkins UI에 직접 작성하지 않고 Git Repository에서 읽는다.

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

<br>

## SCM Checkout

`Pipeline script from SCM`을 사용하면 Jenkins가 기본 Checkout을 수행한다.

```text
Declarative: Checkout SCM
```

특별한 Checkout 처리가 없다면 별도의 `checkout scm` Stage를 중복으로 만들지 않는다.

<br>

## CI에서 Jenkins 역할

```text
Git Push
   ↓
Jenkins
   ├─ Checkout
   ├─ Dependency Install
   ├─ Build
   ├─ Test
   ├─ Container Build
   └─ Registry Push
```

GitOps 환경에서는 Jenkins와 ArgoCD 역할을 분리한다.

```text
GitHub
   ↓
Jenkins
   ├─ Build / Test
   ├─ Container Build
   ├─ ECR Push
   └─ GitOps Image Tag 변경
   ↓
GitOps Repository
   ↓
ArgoCD
   ↓
Kubernetes
```
