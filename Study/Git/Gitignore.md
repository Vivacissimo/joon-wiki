---
title: .gitignore와 이미 추적 중인 파일 제거
description: .gitignore 적용 방법과 이미 GitHub에 올라간 파일 또는 폴더의 Git 추적을 해제하는 방법
published: true
date: 2026-09-02T20:00:00.000+09:00
tags: git, github, gitignore
editor: markdown
dateCreated: 2026-09-02T20:00:00.000+09:00
---

# .gitignore와 이미 추적 중인 파일 제거
<br>

## 1. 핵심

`.gitignore`는 **아직 Git이 추적하지 않는 파일**을 무시할 때 사용한다.

이미 한 번 `git add`, `commit`, `push`까지 해서 GitHub에 올라간 파일은 `.gitignore`에 추가해도 바로 사라지지 않는다.

이 경우에는 `.gitignore`에 추가한 뒤 **Git 추적만 해제**해야 한다.
<br>

## 2. 예시: `.vscode` 폴더가 이미 GitHub에 올라간 경우

`.gitignore`에 아래 내용을 추가한다.

```gitignore
.vscode/
```

이미 추적 중인 `.vscode` 폴더는 아래 명령어로 Git의 인덱스에서만 제거한다.

```bash
git rm -r --cached .vscode
```

`--cached`를 사용하면 **로컬 파일은 삭제하지 않고 Git 추적만 해제**한다.

그다음 변경사항을 커밋하고 원격 저장소에 반영한다.

```bash
git add .gitignore
git commit -m "chore: ignore VS Code settings"
git push origin main
```
<br>

## 3. 전체 흐름

```bash
# 1. .gitignore에 추가
# .vscode/

# 2. 이미 추적 중인 폴더의 Git 추적 해제
git rm -r --cached .vscode

# 3. 상태 확인
git status

# 4. 커밋
git add .gitignore
git commit -m "chore: ignore VS Code settings"

# 5. 원격 저장소 반영
git push origin main
```
<br>

## 4. 파일 하나만 추적 해제할 때

```bash
git rm --cached 파일명
```

예시

```bash
git rm --cached .env
```

로컬 `.env` 파일은 유지하면서 GitHub의 현재 브랜치에서는 제거할 수 있다.
<br>

## 5. 폴더 전체를 추적 해제할 때

```bash
git rm -r --cached 폴더명
```

예시

```bash
git rm -r --cached .vscode
git rm -r --cached node_modules
```

`-r`은 폴더 내부를 재귀적으로 처리한다.
<br>

## 6. `.gitignore`를 추가했는데 계속 추적되는 이유

Git은 한 번 추적하기 시작한 파일을 계속 관리한다.

```text
Untracked File
    ↓ git add
Tracked File
    ↓
.gitignore에 추가해도 계속 Tracked
    ↓
git rm --cached 필요
    ↓
Untracked + Ignored
```
<br>

## 7. 여러 파일의 추적 상태를 한 번에 다시 적용할 때

`.gitignore`를 크게 수정한 뒤 기존 추적 파일을 전체적으로 다시 정리해야 할 때 사용할 수 있다.

```bash
git rm -r --cached .
git add .
git commit -m "chore: apply gitignore"
```

동작 순서

```text
git rm -r --cached .
→ 모든 파일을 Git 인덱스에서 제거

.gitignore 규칙 적용

 git add .
→ 무시 대상이 아닌 파일만 다시 추적
```

프로젝트 전체의 추적 상태가 바뀌므로 필요한 경우에만 사용한다.
<br>

## 8. 확인 방법

```bash
git status
```

`.vscode`가 더 이상 변경 파일이나 Untracked Files에 나타나지 않으면 정상이다.

Git이 특정 파일을 왜 무시하는지 확인할 때는 아래 명령어를 사용할 수 있다.

```bash
git check-ignore -v .vscode/settings.json
```
<br>

## 9. 주의

`git rm --cached`는 **현재 Git 추적과 이후 커밋에서 파일을 제거하는 것**이다.

이미 이전 커밋에 포함된 파일은 Git History에 남아 있다.

따라서 비밀번호, API Key, 인증서 같은 민감 정보가 이미 push된 경우에는 단순히 `.gitignore`와 `git rm --cached`만 사용하면 안 된다.

이 경우에는

1. 노출된 Credential 즉시 폐기 또는 재발급
2. Git History에서 민감 정보 제거
3. `.gitignore`에 해당 파일 추가

순서로 처리한다.
<br>

## 10. 자주 사용하는 `.gitignore` 예시

```gitignore
# VS Code
.vscode/

# Environment
.env
.env.*

# Node.js
node_modules/

# Build
 dist/
build/

# OS
.DS_Store
Thumbs.db
```
<br>

## 11. 정리

```text
아직 Git에 안 올라간 파일
→ .gitignore만 추가

이미 GitHub에 올라간 파일
→ .gitignore 추가
→ git rm --cached
→ commit
→ push

로컬 파일도 같이 삭제
→ git rm
```
