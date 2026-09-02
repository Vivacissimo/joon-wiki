---
title: Git
description: Git과 GitHub의 저장소 구조, 브랜치, 협업 및 배포 전략을 정리하는 상위 문서
published: true
date: 2026-09-02T20:00:00.000+09:00
tags: git, github, devops
editor: markdown
dateCreated: 2026-08-12T00:16:00.000+09:00
---

# Git
<br>

## 1. 개요

**Git**은 소스 코드와 파일의 변경 이력을 관리하는 분산 버전 관리 시스템이다.

실제 프로젝트에서는 Git 명령어뿐만 아니라 저장소를 어떻게 나눌지, 브랜치를 어떻게 운영할지, Pull Request와 CI/CD를 어떤 흐름으로 연결할지도 함께 설계해야 한다.
<br>

## 2. Git 문서 구조

```text
Study/Git/
├── Git.md
├── Gitignore.md
├── Monorepo-Multirepo.md
└── Branch-Strategy.md
```

| 문서 | 내용 |
|---|---|
| [.gitignore와 이미 추적 중인 파일 제거](./Gitignore) | `.gitignore` 적용, `git rm --cached`, 이미 GitHub에 올라간 파일/폴더의 추적 해제 |
| [Monorepo와 Multirepo](./Monorepo-Multirepo) | 하나의 저장소와 여러 저장소 전략의 차이, 장단점, CI/CD 및 GitOps 관점 |
| [Git Branch Strategy](./Branch-Strategy) | Git Flow, GitHub Flow, GitLab Flow, Trunk-Based Development와 브랜치 운영 기준 |
<br>

## 3. Git을 사용할 때 함께 결정해야 하는 것

```text
Git 운영 전략
├── Repository Strategy
│   ├── Monorepo
│   └── Multirepo
│
├── Branch Strategy
│   ├── Git Flow
│   ├── GitHub Flow
│   ├── GitLab Flow
│   └── Trunk-Based Development
│
├── File Tracking
│   ├── .gitignore
│   └── git rm --cached
│
├── Collaboration
│   ├── Pull Request
│   ├── Code Review
│   └── Merge Strategy
│
└── Automation
    ├── CI/CD
    ├── Branch Protection
    └── GitOps
```
<br>

## 4. 앞으로 추가할 수 있는 문서

Git 관련 내용은 이 카테고리 아래에 계속 추가한다.

예시는 아래와 같다.

```text
Study/Git/
├── Git.md
├── Gitignore.md
├── Monorepo-Multirepo.md
├── Branch-Strategy.md
├── Pull-Request.md
├── Merge-Strategy.md
├── Git-Commands.md
├── GitHub-Actions.md
└── GitOps.md
```
<br>

## 5. 참고 자료

- [Git 공식 문서](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com/)
