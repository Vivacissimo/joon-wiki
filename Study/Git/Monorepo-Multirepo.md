---
title: Monorepo와 Multirepo
description: Monorepo와 Multirepo의 차이, 장단점, CI/CD 및 인프라 관점의 선택 기준
published: true
date: 2026-08-11T15:11:00.000Z
tags: git, github, monorepo, multirepo, devops, cicd
editor: markdown
dateCreated: 2026-08-11T15:11:00.000Z
---

# Monorepo와 Multirepo
<br>

## 1. 개요

프로젝트를 Git으로 관리할 때는 코드를 **하나의 Repository에 모을지**, 아니면 **서비스별로 Repository를 분리할지** 결정해야 한다.

대표적인 저장소 전략은 아래 두 가지이다.

```text
Repository Strategy
├── Monorepo
└── Multirepo
```

- **Monorepo**: 여러 애플리케이션, 서비스, 라이브러리를 하나의 Repository에서 관리하는 방식이다.
- **Multirepo**: 애플리케이션이나 서비스 단위로 Repository를 분리해서 관리하는 방식이다.

이 선택은 단순히 Git 폴더를 어떻게 나누는 문제만이 아니다.

아래 항목에 직접 영향을 준다.

- 팀 협업 방식
- 권한 관리
- CI/CD 파이프라인 구성
- 배포 단위
- 코드 리뷰 범위
- 공통 라이브러리 관리
- 서비스 간 의존성 관리
- 장애 영향 범위
- GitOps 구조
<br>

## 2. Monorepo란?

**Monorepo**는 여러 프로젝트나 서비스를 하나의 Git Repository 안에서 관리하는 방식이다.

예를 들어 프론트엔드, 백엔드, AI 서버, 인프라 코드를 하나의 Repository에서 관리할 수 있다.

```text
service-platform/
├── frontend/
├── backend/
├── ai/
├── infra/
├── docs/
└── .github/
    └── workflows/
```

마이크로서비스가 여러 개일 경우에도 아래처럼 구성할 수 있다.

```text
platform/
├── services/
│   ├── user-service/
│   ├── order-service/
│   ├── payment-service/
│   └── notification-service/
│
├── frontend/
├── shared/
├── infra/
└── .github/
```

여기서 중요한 점은 다음과 같다.

> **Monorepo라고 해서 Monolithic Architecture인 것은 아니다.**

Repository가 하나여도 서비스는 각각 독립적인 컨테이너로 빌드하고 독립적으로 배포할 수 있다.

```text
하나의 Git Repository
        ↓
────────────────────────────
Frontend  User  Order  Payment
   ↓       ↓      ↓      ↓
Image    Image   Image   Image
   ↓       ↓      ↓      ↓
독립 배포  독립 배포  독립 배포  독립 배포
```
<br>

## 3. Monorepo의 장점

### 3.1 전체 프로젝트 구조를 한눈에 볼 수 있음

한 Repository에서 전체 시스템 코드를 확인할 수 있다.

```text
frontend
backend
ai
infra
```

서비스 간 관계를 파악하기 쉽고 신규 팀원이 프로젝트를 이해하기에도 편하다.
<br>

### 3.2 공통 코드 관리가 편리함

여러 서비스에서 사용하는 공통 라이브러리나 타입을 같은 Repository 안에서 관리할 수 있다.

```text
shared/
├── types/
├── utils/
└── libraries/
```

공통 모듈을 수정하면서 이를 사용하는 서비스도 같은 Commit 또는 Pull Request에서 함께 수정할 수 있다.
<br>

### 3.3 서비스 간 변경을 하나의 PR로 처리 가능

예를 들어 API 스펙을 변경하면서 프론트엔드와 백엔드를 동시에 수정해야 한다면 하나의 PR로 관리할 수 있다.

```text
PR #120
├── frontend 수정
├── backend API 수정
└── shared type 수정
```

변경의 연관 관계를 하나의 리뷰 단위로 관리하기 쉽다.
<br>

### 3.4 CI/CD 정책을 통합하기 쉬움

하나의 `.github/workflows`에서 여러 서비스의 파이프라인을 관리할 수 있다.

```text
.github/workflows/
├── frontend.yml
├── backend.yml
├── ai.yml
└── infra.yml
```

Path Filter를 이용하면 변경된 서비스만 빌드하거나 배포할 수 있다.

예를 들어 아래와 같은 구조이다.

```yaml
on:
  push:
    paths:
      - 'services/user-service/**'
```

따라서 Monorepo라고 해서 모든 서비스를 항상 같이 빌드해야 하는 것은 아니다.
<br>

## 4. Monorepo의 단점

### 4.1 Repository가 커질 수 있음

서비스와 파일이 계속 증가하면 Repository 크기와 Git history가 커질 수 있다.

대규모 프로젝트에서는 Clone, Checkout, CI 작업 시간이 증가할 수 있으므로 빌드 캐시, sparse checkout, 변경 범위 감지 등의 전략이 필요할 수 있다.
<br>

### 4.2 권한을 세밀하게 분리하기 어려움

Repository 단위 권한 관리가 필요한 경우 Monorepo는 불편할 수 있다.

예를 들어 아래처럼 팀별 접근을 완전히 격리해야 한다면 문제가 될 수 있다.

```text
AI Team       → AI 코드만 접근
Security Team → Security 코드만 접근
Vendor        → Frontend 코드만 접근
```

하나의 Repository 안에서는 기본적으로 Repository 전체에 대한 접근 권한을 가지게 된다.

CODEOWNERS 등을 이용해 리뷰 책임은 나눌 수 있지만 Repository 자체 접근 권한을 완전히 분리하는 것과는 다르다.
<br>

### 4.3 CI 설정이 복잡해질 수 있음

Path Filter를 사용하지 않으면 작은 변경에도 전체 서비스가 빌드될 수 있다.

```text
frontend README 수정
        ↓
Frontend Build
Backend Build
AI Build
Infra Test
```

따라서 규모가 커질수록 변경 파일 기준 실행, 캐시, 병렬 실행 등의 최적화가 중요하다.
<br>

## 5. Multirepo란?

**Multirepo**는 서비스나 애플리케이션 단위로 Git Repository를 분리하는 방식이다.

예를 들어 아래와 같이 관리할 수 있다.

```text
GitHub Organization
│
├── frontend-repo
├── backend-repo
├── ai-repo
├── infra-repo
└── gitops-repo
```

마이크로서비스 환경에서는 다음과 같이 더 세분화할 수도 있다.

```text
Organization
├── user-service
├── order-service
├── payment-service
├── notification-service
├── frontend
└── infrastructure
```
<br>

## 6. Multirepo의 장점

### 6.1 서비스 독립성이 높음

각 서비스가 독립된 Repository를 가지므로 개발, 버전, 릴리즈를 독립적으로 관리할 수 있다.

```text
user-service v1.4.0
payment-service v2.1.0
notification-service v1.0.3
```

서비스별 릴리즈 주기가 다른 조직에서 유리하다.
<br>

### 6.2 권한 관리가 쉬움

Repository 단위로 접근 권한을 분리할 수 있다.

```text
frontend-repo → Frontend Team
ai-repo       → AI Team
infra-repo    → DevOps Team
```

외부 협력사나 별도 조직과 협업할 때 특히 유용하다.
<br>

### 6.3 CI/CD 구성이 단순함

Repository 하나가 하나의 서비스라면 해당 Repository의 변경은 곧 해당 서비스의 변경이다.

```text
payment-service push
        ↓
Payment CI
        ↓
Payment Image Build
        ↓
Payment Deploy
```

Monorepo처럼 변경된 경로를 복잡하게 계산할 필요가 적다.
<br>

### 6.4 장애와 변경의 영향 범위를 분리하기 쉬움

잘못된 Git 설정이나 파이프라인 변경이 다른 Repository에 직접 영향을 주지 않는다.

Repository 단위의 독립성이 높아진다.
<br>

## 7. Multirepo의 단점

### 7.1 Repository 관리 대상이 많아짐

서비스가 증가할수록 Repository도 증가한다.

```text
10 Services
    ↓
10 Repositories
    ↓
10 CI 설정
10 Branch Policy
10 Permission 설정
```

공통 정책을 적용하려면 템플릿이나 중앙 관리 방식이 필요하다.
<br>

### 7.2 서비스 간 변경이 번거로울 수 있음

API 변경으로 여러 서비스가 동시에 수정되어야 하는 경우 각각 PR을 만들어야 할 수 있다.

```text
API 변경
├── frontend PR
├── backend PR
├── api-schema PR
└── deployment PR
```

변경 순서와 호환성 관리가 중요해진다.
<br>

### 7.3 공통 코드 관리가 복잡해질 수 있음

공통 라이브러리를 별도의 Repository 또는 Package로 관리해야 할 수 있다.

```text
shared-library
      ↓
Package Registry
      ↓
Service A
Service B
Service C
```

공통 라이브러리 변경 시 버전 관리가 필요하다.
<br>

## 8. Monorepo와 Multirepo 비교

| 구분 | Monorepo | Multirepo |
|---|---|---|
| Repository 수 | 하나 또는 소수 | 서비스별 여러 개 |
| 전체 구조 파악 | 쉬움 | 상대적으로 어려움 |
| 공통 코드 관리 | 쉬움 | 별도 패키지 관리 필요 |
| 서비스 독립성 | 상대적으로 낮음 | 높음 |
| 권한 분리 | 어려움 | 쉬움 |
| 서비스별 CI/CD | Path Filter 필요 | 자연스럽게 분리 |
| Cross-service 변경 | 하나의 PR 가능 | 여러 PR 필요 가능 |
| Repository 관리 | 단순 | Repository 수만큼 증가 |
| 팀 규모 | 소규모~중간 규모에 편리 | 큰 조직에서 유리할 수 있음 |
| 배포 독립성 | 충분히 가능 | 자연스럽게 가능 |
<br>

## 9. Monorepo와 마이크로서비스의 관계

자주 발생하는 오해가 있다.

```text
Monorepo = Monolith
Multirepo = Microservices
```

이는 잘못된 구분이다.

**Repository 구조와 애플리케이션 아키텍처는 서로 다른 문제이다.**

예를 들어 아래 구조는 Monorepo이면서 Microservices Architecture이다.

```text
platform-repo/
├── user-service/
├── payment-service/
├── order-service/
└── notification-service/
```

각각을 별도 Docker Image로 만들고 Kubernetes에서 독립적인 Deployment로 실행하면 된다.

```text
Git Monorepo
   ↓
CI
   ├── user-service image
   ├── payment-service image
   └── order-service image
        ↓
Container Registry
        ↓
Kubernetes
├── user Deployment
├── payment Deployment
└── order Deployment
```

따라서 **마이크로서비스를 사용한다고 반드시 Multirepo를 사용해야 하는 것은 아니다.**
<br>

## 10. CI/CD 관점에서의 차이

### Monorepo

Monorepo에서는 어떤 서비스가 변경되었는지 판단하는 과정이 중요하다.

```text
Git Push
   ↓
Changed Path 확인
   ↓
services/payment 변경?
   ↓ Yes
Payment Test
   ↓
Payment Build
   ↓
Payment Deploy
```

GitHub Actions의 `paths` 또는 `paths-ignore`를 사용할 수 있다.

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'services/payment/**'
```

또는 변경 파일을 직접 계산하여 Matrix Build를 구성할 수도 있다.
<br>

### Multirepo

Multirepo에서는 Repository 자체가 서비스 경계가 된다.

```text
payment-service Repository
        ↓
Commit
        ↓
Payment CI/CD
```

파이프라인은 단순해지지만 Repository마다 동일한 Workflow를 반복해서 관리해야 할 수 있다.
<br>

## 11. 인프라 Repository는 어떻게 관리해야 하는가?

애플리케이션 Repository 구조와 인프라 Repository 구조는 반드시 동일할 필요가 없다.

예를 들어 애플리케이션은 Monorepo로 운영하면서 인프라는 별도 Repository로 분리할 수 있다.

```text
service-platform
├── frontend
├── backend
└── ai

infra-repo
├── terraform
└── ansible

gitops-repo
├── dev
├── staging
└── prod
```

이런 형태를 **Hybrid Repository Strategy**로 볼 수 있다.
<br>

## 12. GitOps Repository를 별도로 두는 이유

Argo CD와 같은 GitOps 도구를 사용하는 경우 애플리케이션 코드와 Kubernetes 배포 상태를 분리하는 경우가 많다.

예를 들어 다음과 같이 구성한다.

```text
application-repo
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Container Registry
   ↓
GitOps Repository Image Tag 수정
   ↓
Argo CD
   ↓
Kubernetes
```

GitOps Repository는 다음과 같은 내용을 관리한다.

```text
gitops-repo/
├── dev/
│   ├── frontend.yaml
│   └── backend.yaml
├── staging/
└── prod/
```

이렇게 하면 **애플리케이션 코드의 변경과 실제 운영 환경의 상태 변경을 분리해서 추적할 수 있다.**
<br>

## 13. 어떤 전략을 선택해야 하는가?

정답은 없으며 팀과 서비스의 특성에 따라 결정한다.

### Monorepo가 적합한 경우

아래 조건이 많다면 Monorepo가 편리할 가능성이 높다.

- 하나의 제품을 여러 팀이 함께 개발함
- 서비스 간 변경이 자주 발생함
- 공통 코드가 많음
- 팀 규모가 크지 않음
- 전체 코드를 한곳에서 관리하고 싶음
- 통합된 CI/CD 정책을 사용하고 싶음
<br>

### Multirepo가 적합한 경우

아래 조건이 많다면 Multirepo가 적합할 가능성이 높다.

- 서비스별 담당 팀이 명확히 분리됨
- 서비스별 접근 권한을 분리해야 함
- 서비스별 릴리즈 주기가 크게 다름
- 서비스 간 의존성이 낮음
- 각 서비스가 독립적인 제품처럼 운영됨
- 외부 협력사와 일부 서비스만 공유해야 함
<br>

## 14. 실무에서는 Hybrid 방식도 많이 사용함

실제로는 Monorepo와 Multirepo 둘 중 하나만 고집할 필요가 없다.

예를 들어 다음과 같이 구성할 수 있다.

```text
GitHub Organization
│
├── platform-app        ← Monorepo
│   ├── frontend
│   ├── backend
│   └── ai
│
├── infrastructure     ← Terraform / IaC
│
├── gitops             ← Kubernetes Manifest / Helm
│
└── shared-library     ← 공통 라이브러리
```

이 방식은 애플리케이션 개발 편의성과 인프라 및 배포 권한 분리를 동시에 가져갈 수 있다.
<br>

## 15. 프로젝트에서 Repository를 분리할 때 판단 기준

Repository를 만들기 전에 아래 질문을 확인하는 것이 좋다.

```text
1. 서로 같이 변경되는 코드인가?
2. 같은 팀이 관리하는가?
3. 배포 주기가 비슷한가?
4. 권한을 분리해야 하는가?
5. 공통 코드가 많은가?
6. 장애 또는 보안 경계를 분리해야 하는가?
7. 서비스가 독립적으로 운영되는가?
```

예를 들어 다음과 같이 판단할 수 있다.

```text
같은 팀 + 같이 변경 + 공통 코드 많음
                ↓
             Monorepo

독립 팀 + 독립 배포 + 권한 분리 필요
                ↓
             Multirepo
```
<br>

## 16. 정리

Monorepo와 Multirepo의 핵심 차이는 **Repository의 경계를 어디에 둘 것인가**이다.

```text
Monorepo
여러 서비스 → 하나의 Repository

Multirepo
서비스별 → 각각의 Repository
```

Monorepo는 전체 프로젝트의 일관성과 공동 변경에 강점이 있고, Multirepo는 서비스 독립성과 권한 분리에 강점이 있다.

중요한 것은 Repository의 수 자체가 아니라 다음 요소에 맞게 경계를 설정하는 것이다.

- 팀 구조
- 서비스 경계
- 배포 방식
- 권한 구조
- CI/CD 운영 방식
- 변경 빈도

그리고 Repository 구조와 애플리케이션 아키텍처는 별개의 개념이므로 다음 관계를 구분해서 이해해야 한다.

```text
Repository Strategy
Monorepo / Multirepo

Application Architecture
Monolith / Microservices
```

즉, **Monorepo에서도 Microservices Architecture를 충분히 운영할 수 있고, Multirepo라고 해서 반드시 Microservices인 것은 아니다.**
<br>

## 17. 참고 자료

- [GitHub Docs - Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
- [GitHub Docs - Working with monorepos](https://docs.github.com/en/actions/tutorials/build-and-test-code/building-and-testing-nodejs#specifying-a-nodejs-version)
- [GitHub Docs - CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Docs - Repository limits](https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits)
- [Google Research - Why Google Stores Billions of Lines of Code in a Single Repository](https://research.google/pubs/why-google-stores-billions-of-lines-of-code-in-a-single-repository/)
