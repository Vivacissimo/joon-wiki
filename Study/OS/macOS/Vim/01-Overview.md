---
title: 01 - Vim Overview
description: Vim 기본 모드와 실행 방법
published: true
date: 2026-09-24T15:31:00.000Z
tags: vim, macos
editor: markdown
dateCreated: 2026-09-24T15:31:00.000Z
---

# 01. Vim Overview

Vim은 터미널 기반 텍스트 편집기다. 서버에 SSH로 접속해 설정 파일이나 Kubernetes YAML, Shell Script 등을 수정할 때 사용한다.

## 실행

```bash
vim deployment.yaml
vim ~/.vimrc
```

`vi` 입력으로 Vim을 실행하려면 Zsh의 `~/.zshrc`에 `alias vi='vim'`을 추가하고 `source ~/.zshrc`를 실행한다. 이 Alias는 해당 Zsh 세션의 명령어에 적용된다.

## 모드

| 모드 | 역할 | 진입 |
| --- | --- | --- |
| Normal | 이동, 복사, 삭제 등 명령 | `Esc` |
| Insert | 문자 입력 | Normal에서 `i`, `a`, `o` |
| Visual | 범위 선택 | Normal에서 `v`, `V`, `Ctrl+v` |
| Command-line | 저장, 종료, 치환 등 | Normal에서 `:` |

기본 흐름: `vim 파일명` → `i` → 내용 수정 → `Esc` → `:wq`

명령어 모음: [02 - Commands](./02-Commands)

<br>
