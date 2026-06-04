---
title: Git과 연동하기
description: 
published: true
date: 2026-06-04T07:29:19.848Z
tags: 
editor: markdown
dateCreated: 2026-06-04T07:01:01.379Z
---

# Wiki.js GitHub 연동 방법

## 1. 목적

Wiki.js에 작성한 문서를 GitHub 저장소와 동기화하여 문서 내용을 안전하게 백업한다.

라즈베리파이 장애, SD카드 손상, Docker 볼륨 삭제 등의 상황이 발생해도 GitHub에 저장된 Markdown 문서를 기준으로 복구할 수 있다.

```txt
Wiki.js
  ↓
Git Storage
  ↓
GitHub Repository
```

---

## 2. GitHub 저장소 생성

GitHub에서 Wiki.js 문서를 저장할 새 저장소를 생성한다.

추천 설정:

```txt
Repository name: joon-wiki
Visibility: Private
README: 생성하지 않음
.gitignore: 생성하지 않음
License: 생성하지 않음
```

저장소는 Wiki.js 전용으로 사용하는 것을 권장한다.

예시 저장소 주소:

```txt
git@github.com:본인계정/joon-wiki.git
```

---

## 3. 라즈베리파이에서 SSH 키 생성

라즈베리파이에 접속한다.

```bash
ssh joon@192.168.35.200
```

Wiki.js 작업 폴더로 이동한다.

```bash
cd ~/services/wikijs
```

SSH 키 저장 폴더를 만든다.

```bash
mkdir -p keys
```

GitHub 연동용 SSH 키를 생성한다.

```bash
ssh-keygen -t rsa -b 4096 -f ~/services/wikijs/keys/wiki_github -N ""
```

생성 확인:

```bash
ls -l ~/services/wikijs/keys
```

정상이라면 아래 두 파일이 생성된다.

```txt
wiki_github
wiki_github.pub
```

파일 의미:

```txt
wiki_github      → Private Key, 외부 공개 금지
wiki_github.pub  → Public Key, GitHub에 등록
```

공개키 내용을 확인한다.

```bash
cat ~/services/wikijs/keys/wiki_github.pub
```

출력된 내용을 복사해둔다.



## 4. GitHub Deploy Key 등록

GitHub 저장소에서 아래 메뉴로 이동한다.

```txt
Repository
→ Settings
→ Deploy keys
→ Add deploy key
```

입력값:

```txt
Title:
wikijs-raspberrypi

Key:
방금 복사한 wiki_github.pub 내용

Allow write access:
체크
```

`Allow write access`를 체크해야 Wiki.js가 GitHub 저장소로 문서를 push할 수 있다.

---

## 5. Wiki.js 컨테이너에 SSH 키 연결

현재 Docker Compose 파일을 연다.

```bash
cd ~/services/wikijs
nano docker-compose.yml
```

`wiki` 서비스 아래에 `volumes` 항목을 추가한다.

```yaml
services:
  db:
    image: postgres:15-alpine
    container_name: wikijs-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    container_name: wikijs
    restart: unless-stopped
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: ${POSTGRES_USER}
      DB_PASS: ${POSTGRES_PASSWORD}
      DB_NAME: ${POSTGRES_DB}
    ports:
      - "3000:3000"
    volumes:
      - ./keys/wiki_github:/etc/wiki/github.pem:ro

volumes:
  db_data:
```

중요한 부분:

```yaml
volumes:
  - ./keys/wiki_github:/etc/wiki/github.pem:ro
```

의미:

```txt
라즈베리파이 파일:
~/services/wikijs/keys/wiki_github

컨테이너 내부 경로:
/etc/wiki/github.pem

권한:
read only
```

Compose 설정을 다시 적용한다.

```bash
docker compose up -d
```

컨테이너 상태를 확인한다.

```bash
docker compose ps
```

---

## 6. GitHub SSH 접속 테스트

Wiki.js 컨테이너 안으로 들어간다.

```bash
docker exec -it wikijs sh
```

컨테이너 내부에서 SSH 키 파일이 보이는지 확인한다.

```bash
ls -l /etc/wiki/github.pem
```

GitHub 접속을 테스트한다.

```bash
ssh -i /etc/wiki/github.pem -T git@github.com
```

처음 접속이면 아래처럼 물어볼 수 있다.

```txt
Are you sure you want to continue connecting?
```

`yes`를 입력한다.

정상이라면 아래와 비슷한 메시지가 나온다.

```txt
Hi 계정명! You've successfully authenticated, but GitHub does not provide shell access.
```

컨테이너에서 나온다.

```bash
exit
```

---

## 7. Wiki.js 관리자 페이지에서 Git Storage 설정

브라우저에서 Wiki.js에 접속한다.

```txt
http://192.168.35.200:3000
```

관리자 메뉴로 이동한다.

```txt
Administration
→ Storage
→ Git
```

Git Storage를 활성화한다.

설정값 예시:

```txt
Authentication Type:
ssh

# ssh 방식이라 ssh 주소로 
Repository URI:
git@github.com:본인계정/joon-wiki.git

Branch:
main

SSH Private Key Path:
/etc/wiki/github.pem

Default Author Email:
본인 GitHub 이메일

Default Author Name:
본인 이름

Local Repository Path:
./data/repo

Verify SSL Certificate:
On

Sync Direction:
Bi-directional

Sync Schedule:
5 minutes

# 안 적혀있는 건 안 넣어도 됨
```

권장 설정:

```txt
Sync Direction:
Bi-directional
```

의미:

```txt
Wiki.js에서 작성한 문서 → GitHub로 push
GitHub에서 변경한 문서 → Wiki.js로 pull
```

설정 후 오른쪽 위 `Apply`를 누른다.

---

## 8. 기존 문서를 GitHub에 올리기

이미 Wiki.js에 작성한 문서가 있다면 바로 GitHub에 올라가지 않을 수 있다.

이 경우 아래 메뉴를 사용한다.

```txt
Administration
→ Storage
→ Git
→ Actions
→ Add Untracked Changes
→ Force Sync
```

이후 GitHub 저장소에 Markdown 파일이 올라왔는지 확인한다.

---

## 9. 동작 확인

Wiki.js에서 테스트 페이지를 하나 만든다.

```markdown
# Git 연동 테스트

이 문서는 Wiki.js에서 작성했고 GitHub로 동기화되는지 확인하기 위한 테스트 문서입니다.
```

저장 후 몇 분 기다린다.

그 다음 GitHub 저장소를 확인한다.

정상이라면 Wiki.js에서 작성한 문서가 GitHub 저장소에 Markdown 파일로 올라와야 한다.

---

## 10. 문제 해결

### 10-1. GitHub에 push가 안 되는 경우

Deploy Key에 쓰기 권한이 없는 경우가 많다.

GitHub 저장소에서 아래 메뉴를 확인한다.

```txt
Repository
→ Settings
→ Deploy keys
→ wikijs-raspberrypi
```

`Allow write access`가 체크되어 있어야 한다.

---

### 10-2. SSH 키 경로 오류

Wiki.js 설정의 SSH 키 경로가 정확해야 한다.

```txt
/etc/wiki/github.pem
```

컨테이너 내부에서 확인한다.

```bash
docker exec -it wikijs sh
ls -l /etc/wiki/github.pem
exit
```

---

### 10-3. 저장소 주소 오류

GitHub 저장소 주소는 HTTPS가 아니라 SSH 주소를 사용한다.

올바른 예:

```txt
git@github.com:본인계정/joon-wiki.git
```

잘못된 예:

```txt
https://github.com/본인계정/joon-wiki.git
```

SSH 인증을 사용할 것이므로 `git@github.com:...` 형식으로 입력한다.

---

### 10-4. 브랜치 이름 오류

GitHub 저장소 기본 브랜치가 `main`인지 확인한다.

```txt
Branch:
main
```

만약 저장소 기본 브랜치가 `master`라면 Wiki.js 설정도 `master`로 맞춘다.

---

## 11. 최종 점검 명령어

컨테이너 상태 확인:

```bash
cd ~/services/wikijs
docker compose ps
```

Wiki.js 로그 확인:

```bash
docker compose logs -f wiki
```

GitHub SSH 테스트:

```bash
docker exec -it wikijs sh
ssh -i /etc/wiki/github.pem -T git@github.com
exit
```

**만약 한번도 git에 올린적이 없다면**
- main 브랜치 없어서 싱크 안될수도 있음
- git repo에서 README.md 그냥 파일 만들기
- main브랜치와 리드미 파일 생성 됐는지 확인
- 다시 wiki 세팅 들어가서 apply하기
- ACTIONS -> Add Untracked Changes -> Force Sync


---

## 12. 정리

```txt
Wiki.js GitHub 연동은 문서 백업용이다.
GitHub Deploy Key는 write access가 필요하다.
Wiki.js 컨테이너에서 SSH Private Key를 읽을 수 있어야 한다.
Repository URI는 SSH 형식으로 입력한다.
Sync Direction은 Bi-directional로 설정하면 양방향 동기화가 가능하다.
```