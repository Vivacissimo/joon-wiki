---
title: Git Branch Strategy
description: Git Flow, GitHub Flow, GitLab Flow, Trunk-Based Development와 브랜치 운영 기준
published: true
date: 2026-08-12T00:14:00.000+09:00
tags: git, github, branch, git-flow, github-flow, devops
editor: markdown
dateCreated: 2026-08-12T00:14:00.000+09:00
---

# Git Branch Strategy
<br>

## 1. 브랜치 전략이란?

**브랜치 전략(Branching Strategy)**은 팀이 Git 브랜치를 어떤 기준으로 만들고, 어떤 브랜치에 병합하며, 언제 삭제할지 정한 운영 규칙이다.

브랜치 전략을 사용하는 이유는 여러 개발자가 동시에 작업하더라도 안정적인 코드를 보호하고 개발, 테스트, 배포 단계를 구분하기 위해서이다.

```text
개발
 ↓
검증
 ↓
병합
 ↓
배포
```

브랜치 전략은 위 과정을 Git Branch로 표현한 것이라고 볼 수 있다.
<br>

## 2. 브랜치 전략이 필요한 이유

여러 개발자가 모두 `main` 브랜치에서 직접 작업하면 미완성 기능이나 오류가 그대로 안정 버전에 포함될 수 있다.

```text
main
├── 로그인 개발 중
├── 결제 수정 중
├── AI 기능 개발 중
└── 인프라 설정 변경 중
```

작업 브랜치를 분리하면 기능별 변경을 격리할 수 있다.

```text
main
  ↑
 dev
  ├── feature/login
  ├── feature/payment
  ├── feature/ai
  └── fix/api-error
```

주요 목적은 아래와 같다.

- 안정적인 `main` 보호
- 여러 개발자의 병렬 작업
- 기능별 변경 이력 분리
- Pull Request 기반 코드 리뷰
- CI 테스트 후 병합
- 개발/운영 환경 분리
- 장애 발생 시 변경 추적 및 롤백
<br>

## 3. 브랜치 이름 예시

| 브랜치 | 역할 |
|---|---|
| `main` | 운영 가능한 안정 버전 |
| `dev` 또는 `develop` | 기능을 통합하는 개발 브랜치 |
| `feature/*` | 신규 기능 개발 |
| `fix/*` | 일반 버그 수정 |
| `hotfix/*` | 운영 환경 긴급 수정 |
| `release/*` | 배포 전 최종 안정화 |
| `chore/*` | 설정, 문서, 빌드 등 기능 외 작업 |

예시는 아래와 같다.

```text
feature/login
feature/image-convert
fix/upload-error
hotfix/payment-error
release/1.2.0
chore/update-actions
```
<br>

## 4. Git Flow

Git Flow는 `main`과 `develop`을 중심으로 여러 종류의 보조 브랜치를 사용하는 전략이다.

```text
main
 ↑
release/* ← develop
              ↑
        feature/*

hotfix/* → main
       └→ develop
```

주요 브랜치는 아래와 같다.

| 브랜치 | 역할 |
|---|---|
| `main` | 실제 릴리즈된 코드 |
| `develop` | 다음 릴리즈를 위한 통합 개발 |
| `feature/*` | 기능 개발 |
| `release/*` | 배포 전 테스트와 버전 준비 |
| `hotfix/*` | 운영 긴급 수정 |

### 일반 흐름

```text
feature/login
      ↓
develop
      ↓
release/1.0.0
      ↓
main
```

운영 장애가 발생하면 아래처럼 처리할 수 있다.

```text
main
 ↓
hotfix/login-error
 ↓
main + develop
```

### 장점

- 개발과 운영의 경계가 명확하다.
- 릴리즈 준비 단계를 별도로 관리하기 쉽다.
- 정기적인 버전 릴리즈를 하는 프로젝트에 적합하다.

### 단점

- 브랜치 종류가 많다.
- Merge 단계가 많아진다.
- 자주 배포하는 CI/CD 환경에서는 과하게 복잡할 수 있다.
<br>

## 5. GitHub Flow

GitHub Flow는 `main`과 짧게 사용하는 작업 브랜치를 중심으로 하는 단순한 전략이다.

```text
main
├── feature/login
├── feature/payment
└── fix/api-error
```

일반적인 흐름은 아래와 같다.

```text
main
 ↓
feature/login
 ↓
Pull Request
 ↓
Review + CI
 ↓
main
 ↓
Deploy
```

별도의 `develop`이나 `release` 브랜치를 필수로 두지 않는다.

### 장점

- 구조가 단순하다.
- Pull Request 중심 협업과 잘 맞는다.
- CI/CD를 이용해 자주 배포하는 서비스에 적합하다.

### 단점

- `main`에 병합되는 코드 품질을 CI와 리뷰가 강하게 보장해야 한다.
- 개발 환경과 운영 환경을 브랜치로 명확하게 분리하려는 팀에는 추가 규칙이 필요할 수 있다.
<br>

## 6. GitLab Flow

GitLab Flow는 GitHub Flow의 단순함을 유지하면서 환경 또는 릴리즈 브랜치를 추가하는 방식으로 이해할 수 있다.

예를 들어 환경별 브랜치를 사용할 수 있다.

```text
feature/*
    ↓
main
    ↓
staging
    ↓
production
```

또는 팀 운영 방식에 따라 아래처럼 구성할 수도 있다.

```text
feature/*
    ↓
dev
    ↓
staging
    ↓
main
```

환경별 배포 단계가 명확한 프로젝트에서 사용하기 쉽다.
<br>

## 7. Trunk-Based Development

Trunk-Based Development는 모든 개발자가 `main`과 같은 하나의 중심 브랜치인 **Trunk**에 자주 작은 변경을 병합하는 방식이다.

```text
main
 ↑ ↑ ↑ ↑
짧은 작업 브랜치
```

작업 브랜치를 사용하더라도 오래 유지하지 않고 빠르게 병합한다.

```text
feature/a ─┐
feature/b ─┼→ main → CI → Deploy
feature/c ─┘
```

### 특징

- 작은 단위로 자주 Commit한다.
- 장기간 유지되는 Feature Branch를 피한다.
- 강력한 자동 테스트와 CI가 중요하다.
- 미완성 기능은 Feature Flag로 숨길 수 있다.

### 장점

- Merge Conflict가 줄어든다.
- Continuous Integration과 잘 맞는다.
- 배포 주기를 짧게 가져갈 수 있다.

### 단점

- 테스트 자동화 수준이 낮으면 위험하다.
- 팀의 CI/CD 성숙도가 필요하다.
<br>

## 8. 전략 비교

| 구분 | Git Flow | GitHub Flow | GitLab Flow | Trunk-Based |
|---|---|---|---|---|
| 구조 | 복잡 | 단순 | 중간 | 매우 단순 |
| `develop` | 사용 | 보통 미사용 | 선택 | 미사용 |
| `release` | 사용 | 보통 미사용 | 선택 | 미사용 |
| 배포 주기 | 비교적 긴 주기 | 짧은 주기 | 환경에 따라 다름 | 매우 짧은 주기 |
| CI/CD 적합성 | 보통 | 높음 | 높음 | 매우 높음 |
| 운영 난이도 | 높음 | 낮음 | 중간 | 자동화 수준에 따라 다름 |
<br>

## 9. 소규모 팀에서 사용할 수 있는 전략

프로젝트 규모가 크지 않고 개발 환경과 운영 환경을 구분하고 싶다면 아래 정도의 구조가 이해하기 쉽다.

```text
main
 │
 dev
 │
 ├── feature/*
 ├── fix/*
 └── chore/*
```

기능 개발 흐름은 아래와 같다.

```text
1. dev에서 feature 브랜치 생성
2. feature에서 기능 개발
3. feature → dev Pull Request
4. CI 및 코드 리뷰
5. dev에서 통합 테스트
6. dev → main Pull Request
7. main 병합 후 운영 배포
```

```text
feature/login
      ↓ PR
     dev
      ↓ Integration Test
     main
      ↓
 Production
```

이 방식은 Git Flow를 단순화한 형태로 볼 수 있으며, `release/*` 브랜치가 꼭 필요하지 않은 프로젝트에서 사용하기 편하다.
<br>

## 10. CI/CD와 브랜치 전략

브랜치 전략은 CI/CD와 함께 설계하는 것이 좋다.

예를 들어 아래와 같이 역할을 나눌 수 있다.

```text
feature/*
   ↓
Lint + Unit Test

   ↓ PR

dev
   ↓
Build + Integration Test + Dev Deploy

   ↓ PR

main
   ↓
Build + Production Deploy
```

즉 브랜치 자체보다 중요한 것은 **각 브랜치에서 어떤 검증과 배포가 실행되는지**이다.
<br>

## 11. Branch Protection

`main`이나 `dev`처럼 중요한 브랜치에는 직접 Push를 제한하고 Pull Request를 통해서만 병합하도록 설정할 수 있다.

예를 들어 `main`에 아래 정책을 적용할 수 있다.

- Pull Request 필수
- 최소 1명 이상 Review 필수
- CI 성공 필수
- 최신 브랜치 상태 요구
- 직접 Push 제한
- Force Push 제한

```text
Developer
   X
   │ direct push
   ↓
 main

Developer
   ↓
feature
   ↓ PR
Review + CI
   ↓
 main
```
<br>

## 12. Merge 방식

Pull Request를 병합할 때는 대표적으로 세 가지 방식을 사용할 수 있다.

| 방식 | 특징 |
|---|---|
| Merge Commit | 기존 Commit과 Branch 흐름을 그대로 보존 |
| Squash Merge | PR의 여러 Commit을 하나로 합쳐 병합 |
| Rebase Merge | Commit을 기준 브랜치 위에 재배치하여 선형 History 생성 |

소규모 프로젝트에서는 Feature Branch의 작업 Commit이 많을 경우 **Squash Merge**를 사용하면 `main` History를 비교적 깔끔하게 유지할 수 있다.
<br>

## 13. 브랜치 전략 선택 기준

브랜치 전략은 복잡할수록 좋은 것이 아니다.

아래 항목을 기준으로 선택한다.

| 질문 | 고려할 전략 |
|---|---|
| 릴리즈 버전을 장기간 준비하는가? | Git Flow |
| 웹 서비스를 자주 배포하는가? | GitHub Flow |
| Dev/Staging/Prod 환경 흐름이 중요한가? | GitLab Flow 또는 환경 브랜치 전략 |
| 하루에도 여러 번 배포하는가? | Trunk-Based Development |
| 소규모 팀이고 구조를 단순하게 유지하고 싶은가? | GitHub Flow 또는 `main + dev + feature` |
<br>

## 14. 핵심 정리

```text
Git Branch Strategy
= 코드를 안전하게 개발하고 검증한 뒤
  안정적인 브랜치에 병합하기 위한 운영 규칙
```

브랜치의 개수가 중요한 것이 아니라 아래 세 가지가 명확해야 한다.

1. 어디에서 기능을 개발하는가
2. 어떤 검증을 거쳐 병합하는가
3. 어느 브랜치가 실제 배포 기준인가
<br>

## 15. 참고 자료

- [Git 공식 문서 - Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [GitHub Docs - GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [GitHub Docs - About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitLab Docs - GitLab Flow](https://docs.gitlab.com/topics/gitlab_flow/)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
