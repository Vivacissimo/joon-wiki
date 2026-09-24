---
title: 02 - iTerm2 설정 Git으로 관리
description: iTerm2 설정을 dotfiles 저장소로 백업하고 다른 Mac에 적용
published: true
date: 2026-09-24T14:30:00.000Z
tags: macos, iterm2, git, dotfiles
editor: markdown
dateCreated: 2026-09-24T14:30:00.000Z
---

# 02. iTerm2 설정 Git으로 관리

## 1. 목적

iTerm2의 설정 폴더를 Git 저장소에 넣어 백업한다. 한 Mac에서 설정을 변경해 Push한 뒤 다른 Mac에서 Pull하여 같은 환경을 사용한다.

```text
Mac A - iTerm2 설정 변경
  → 사용자 지정 폴더에 저장
  → Git commit / push
  → GitHub dotfiles
  → Mac B - Git pull
  → iTerm2 설정 다시 불러오기
```

<br>

## 2. 저장소 준비

iTerm2 외에 Zsh와 Git 설정까지 관리할 수 있도록 `dotfiles`라는 이름을 사용한다.

```bash
mkdir -p ~/dotfiles/iterm2
cd ~/dotfiles
git init
```

```text
dotfiles/
├── iterm2/
│   └── com.googlecode.iterm2.plist
├── zsh/
│   └── .zshrc
├── git/
│   └── .gitconfig
└── README.md
```

위 구조에서 iTerm2 폴더를 먼저 만들고, 나머지는 실제로 관리할 때 추가한다. 생성 전의 예시 파일까지 미리 만들 필요는 없다.

<br>

## 3. iTerm2 설정 폴더 지정

iTerm2에서 다음 메뉴로 이동한다.

`iTerm2 → Settings(또는 Preferences) → General → Settings`

1. **Load settings from a custom folder or URL** 활성화
2. 폴더를 `~/dotfiles/iterm2`로 지정한다. 폴더 선택 창에서는 실제 사용자 홈 디렉토리의 경로를 선택한다.
3. **Save changes to folder when iTerm2 quits** 활성화
4. 설정을 외부 폴더에 복사하거나 저장할 것인지 묻는 경우 저장한다.
5. iTerm2를 종료한 뒤 파일이 생성되었는지 확인한다.

```bash
ls -la ~/dotfiles/iterm2
```

예상 파일:

```text
com.googlecode.iterm2.plist
```

사용 중인 iTerm2 버전이나 옵션에 따라 메뉴 문구가 조금 다를 수 있다.

<br>

## 4. GitHub에 백업

GitHub에 `dotfiles` 저장소를 생성한 뒤 연결한다. 아래 주소는 저장소 이름이 `dotfiles`일 때의 예시다.

```bash
cd ~/dotfiles

git add iterm2/
git commit -m "chore: add iterm2 settings"

git branch -M main
git remote add origin https://github.com/Vivacissimo/dotfiles.git
git push -u origin main
```

기존에 `origin`이 있으면 `git remote add`를 다시 실행하지 않는다. `git remote -v`로 먼저 확인한다.

### 업로드 전 확인

프로필에는 SSH 호스트, 사용자명, 실행 명령어 등 개인 환경 정보가 들어 있을 수 있다.

```bash
git diff --cached
```

처음에는 Private 저장소로 관리한다. 비밀번호, 토큰, 개인 SSH 키는 커밋하지 않는다.

<br>

## 5. 다른 Mac에서 가져오기

```bash
cd ~
git clone https://github.com/Vivacissimo/dotfiles.git
```

새 Mac에 iTerm2를 설치하고 `Settings → General → Settings`에서 Clone한 `~/dotfiles/iterm2` 폴더를 지정한다. 기존 설정을 덮어쓸 수 있으므로 새 Mac의 설정이 필요하다면 먼저 백업한다.

외부 폴더의 설정을 불러온 뒤 iTerm2를 재시작한다. 폰트나 컬러 프리셋 등 별도 설치가 필요한 항목은 새 Mac에도 설치한다.

<br>

## 6. 평소 동기화

### 작업 시작 전

```bash
cd ~/dotfiles
git pull --ff-only
```

### 설정 변경 후

iTerm2를 종료하여 설정 파일을 저장한다.

```bash
cd ~/dotfiles
git status
git add iterm2/
git commit -m "chore: update iterm2 settings"
git push
```

### 다른 Mac에서

```bash
cd ~/dotfiles
git pull --ff-only
```

iTerm2를 다시 실행하여 가져온 설정을 적용한다. 양쪽 Mac에서 같은 plist 파일을 동시에 수정하면 Git 충돌이 발생할 수 있다. 설정 파일은 자동 병합보다 어느 쪽 설정을 사용할지 확인한 뒤 처리한다.

**Git Push/Pull은 자동으로 실행되지 않는다.** iTerm2의 외부 폴더 저장과 Git 동기화는 별개의 동작이다.

<br>

## 7. iTerm2와 Shell 설정 구분

iTerm2 설정은 터미널 앱의 프로필·UI 동작에 관한 것이다. Zsh alias와 PATH는 별도 설정 파일에서 관리한다.

```text
iTerm2 (터미널 앱)
  └── Zsh (셸)
       ├── .zshrc
       └── .zprofile
```

`.zshrc`, `.gitconfig`를 Git에 포함할 때는 API 키나 회사 내부 주소 등 공개하면 안 되는 내용이 있는지 별도로 확인한다.

<br>

## 참고

- [iTerm2 공식 문서 - Preferences](https://iterm2.com/documentation-preferences-general.html)
- [iTerm2 공식 홈페이지](https://iterm2.com/)
- [Git 공식 문서](https://git-scm.com/doc)

<br>
