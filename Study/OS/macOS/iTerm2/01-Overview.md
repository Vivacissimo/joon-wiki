---
title: 01 - iTerm2 Overview
description: iTerm2 개념과 설정 관리 범위
published: true
date: 2026-09-24T14:30:00.000Z
tags: macos, iterm2
editor: markdown
dateCreated: 2026-09-24T14:30:00.000Z
---

# 01. iTerm2 Overview

iTerm2는 macOS에서 사용하는 터미널 에뮬레이터다. 프로필, 색상, 단축키, 창 분할 등 터미널 앱의 동작을 설정할 수 있다.

## Git으로 관리할 수 있는 것

iTerm2는 설정을 사용자 지정 폴더에 저장할 수 있다. 해당 폴더를 Git 저장소에 포함하면 여러 Mac에서 설정을 공유할 수 있다.

| 구분 | 관리 위치 |
| --- | --- |
| iTerm2 프로필·색상·단축키 | `com.googlecode.iterm2.plist` |
| Zsh alias·플러그인 설정 | `~/.zshrc` |
| Zsh 로그인 환경변수 | `~/.zprofile` |
| Git 설정 | `~/.gitconfig` |

iTerm2 설정 파일만으로 Zsh 플러그인이나 외부 폰트의 설치까지 복원되는 것은 아니다.

다음 문서: [02 - Git으로 설정 관리](./02-Git-Settings)

<br>
