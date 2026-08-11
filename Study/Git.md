---
title: Git
description: Git, GitHub, 저장소 구조와 협업 전략을 정리하는 카테고리
authorized: true
published: true
date: 2026-08-11T15:08:00.000Z
tags: git, github, devops
editor: markdown
dateCreated: 2026-08-11T15:08:00.000Z
---

# Git
<br>

## 1. Git이란?

**Git**은 소스 코드와 파일의 변경 이력을 기록하고 여러 사람이 동시에 작업할 수 있도록 도와주는 **분산 버전 관리 시스템(Distributed Version Control System)**이다.

Git을 사용하면 단순히 코드를 저장하는 것뿐만 아니라 아래와 같은 개발 과정 전체를 관리할 수 있다.

- 변경 이력 관리
- 브랜치를 이용한 기능 개발
- 코드 병합
- 협업 및 코드 리뷰
- 릴리즈 버전 관리
- CI/CD 파이프라인 연동

```text
개발자 작업
   ↓
Git Commit
   ↓
Branch
   ↓
Pull Request
   ↓
Review / CI
   ↓
Merge
   ↓
Deploy
```
<br>

## 2. Git 카테고리에서 다루는 내용

이 카테고리에서는 Git 명령어 자체뿐만 아니라 실제 프로젝트에서 Git과 GitHub를 어떻게 운영할지에 대한 내용을 함께 정리한다.

| 구분 | 주요 내용 |
|---|---|
| Repository | 저장소 구성, Monorepo, Multirepo |
| Branch | 브랜치 전략, Git Flow, GitHub Flow |
| Collaboration | Pull Request, Code Review, Merge |
| CI/CD | GitHub Actions, Path Filter, 배포 연동 |
| Release | Tag, Version, Release 관리 |
| GitOps | Git을 기준으로 인프라 및 배포 상태 관리 |
<br>

## 3. Repository Strategy

프로젝트를 Git으로 관리할 때는 먼저 **서비스와 코드를 몇 개의 Repository로 나눌 것인지** 결정해야 한다.

대표적인 방식은 아래 두 가지이다.

```text
Repository Strategy
├── Monorepo
└── Multirepo
```

- **Monorepo**: 여러 애플리케이션이나 서비스를 하나의 Repository에서 관리한다.
- **Multirepo**: 애플리케이션이나 서비스별로 Repository를 분리해서 관리한다.

두 방식은 프로젝트 규모, 팀 구조, 서비스 간 의존성, 배포 방식, 권한 관리 방식에 따라 장단점이 달라진다.

자세한 내용은 아래 문서에서 정리한다.

- [Monorepo와 Multirepo](./Git/Monorepo-Multirepo)
<br>

## 4. 참고 자료

- [Git 공식 문서](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com/)
