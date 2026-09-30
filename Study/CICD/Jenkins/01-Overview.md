---
title: Jenkins Overview
description: Jenkins 개념과 기본 구성
published: true
tags: jenkins,cicd,devops
editor: markdown
---

# Jenkins Overview

## 개념

Jenkins는 CI/CD Pipeline을 구성하는 오픈소스 자동화 서버이다.

- Build
- Test
- Container Image Build
- Registry Push
- 배포 자동화
- Plugin 기반 기능 확장

```text
Code Push
   ↓
Jenkins
   ├─ Checkout
   ├─ Build
   ├─ Test
   └─ Image Build
```

<br>

## 구성

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

<br>

## Job

Jenkins에서 실행할 작업 단위이다.

- Freestyle Project
- Pipeline
- Multibranch Pipeline

Pipeline은 Build, Test, Image Build 같은 전체 흐름을 코드로 관리한다.

<br>

## Plugin

Jenkins 기능 확장 단위이다.

예시

- Git Plugin
- GitHub Plugin
- NodeJS Plugin
- Docker Pipeline Plugin
- Kubernetes Plugin
- Credentials Plugin

<br>

## Tool 관리

Build에 필요한 Runtime과 Tool을 Jenkins에서 관리할 수 있다.

- JDK
- Maven
- Gradle
- Node.js

```groovy
tools {
    nodejs 'node22'
}
```
