---
title: domain 연결하기
description: 
published: true
date: 2026-06-04T10:39:12.387Z
tags: 
editor: markdown
dateCreated: 2026-06-04T10:39:12.387Z
---

# `wiki.junu.dev` → Raspberry Pi 5 Wiki.js 연결

## 1. 최종 목표

```text
https://wiki.junu.dev
        ↓
Cloudflare
        ↓
Cloudflare Tunnel
        ↓
Raspberry Pi
        ↓
http://127.0.0.1:3000
        ↓
Wiki.js
```

 **공유기 포트포워딩 없이** `wiki.junu.dev` 도메인으로 Raspberry Pi의 Wiki.js에 접속하는 방식이다.

<br>

## 2. 전체 구조

```text
사용자 브라우저
        ↓
https://wiki.junu.dev
        ↓
Cloudflare DNS / Proxy
        ↓
Cloudflare Tunnel
        ↓
Raspberry Pi cloudflared
        ↓
http://127.0.0.1:3000
        ↓
Wiki.js Container
        ↓
PostgreSQL Container
```
<br>

## 3. 사용한 방식

보안을 위해 일반적인 포트포워딩 방식이 아니라 **Cloudflare Tunnel**을 사용했다.

### 사용하지 않은 방식

```text
공유기 80 포트 오픈  ❌
공유기 443 포트 오픈 ❌
공유기 3000 포트 오픈 ❌
공유기 5432 포트 오픈 ❌
```

### 사용한 방식

```text
외부 사용자
    ↓
wiki.junu.dev
    ↓
Cloudflare
    ↓
Cloudflare Tunnel
    ↓
Raspberry Pi 내부 서비스
    ↓
Wiki.js
```
<br>

## 4. 기존 Wiki.js 상태 확인

Raspberry Pi에서 기존 Wiki.js 컨테이너 상태를 확인했다.

```bash
docker ps -a --filter "name=wikijs"
```

확인된 컨테이너는 결과

```text
wikijs      → ghcr.io/requarks/wiki:2
wikijs-db   → postgres:15-alpine
```

Wiki.js가 로컬에서 정상 동작하는지도 확인

```bash
curl -I http://127.0.0.1:3000
```


```text
HTTP/1.1 200 OK
```

Wiki.js 자체는 정상 실행 중
<br>

## 5. 기존 데이터 보호

기존 Wiki.js에 이미 작성한 글이 있었기 때문에 DB 컨테이너를 삭제하지 않는 방식으로 접근

### 절대 하면 안 되는 명령어

```bash
docker rm -f wikijs-db
docker compose down -v
docker volume rm ...
docker system prune --volumes
```

특히 아래 명령어는 DB 볼륨까지 삭제될 수 있으므로 주의해야 한다.

```bash
docker compose down -v
```

`-v` 옵션은 Docker Compose에서 사용하는 볼륨까지 제거할 수 있기 때문에, PostgreSQL 데이터가 사라질 수 있다.

---

## 6. Wiki.js DB 백업

기존 데이터를 보호하기 위해 PostgreSQL 백업을 먼저 생성했다.

```bash
mkdir -p /srv/junu/wiki/backup
```

```bash
docker exec -t wikijs-db sh -lc 'pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB"' > /srv/junu/wiki/backup/wikijs_$(date +%F_%H%M).sql
```

생성된 백업 파일은 다음과 같다.

```text
/srv/junu/wiki/backup/wikijs_2026-06-04_1854.sql
```

백업 파일 확인 명령어는 다음과 같다.

```bash
ls -lh /srv/junu/wiki/backup
```
<br>


## 7. Docker Compose 위치 확인

처음에는 `/srv/junu/wiki` 경로에서 새로 `docker compose up -d`를 실행하려고 했지만, 기존 컨테이너 이름과 충돌이 발생했다.

오류 내용:

```text
Conflict. The container name "/wikijs-db" is already in use
```

기존 Wiki.js가 실제로 관리되는 Docker Compose 경로를 확인했다.

```bash
docker inspect wikijs-db --format '{{ index .Config.Labels "com.docker.compose.project.working_dir" }}'
```

결과:

```text
/home/joon/services/wikijs
```

따라서 앞으로 Wiki.js를 관리할 때는 아래 경로에서 작업해야 한다.

```bash
cd /home/joon/services/wikijs
```
<br>

## 8. Docker 포트 보안 수정

처음에는 Wiki.js 포트가 다음과 같이 열려 있었다.

```text
0.0.0.0:3000->3000/tcp
```

이 상태는 Raspberry Pi의 모든 네트워크 인터페이스에서 `3000`번 포트를 받을 수 있다는 뜻이다.

즉, 같은 네트워크나 외부 설정에 따라 `집IP:3000` 형태로 직접 접근될 가능성이 있다.

보안을 위해 `docker-compose.yml`에서 Wiki.js 포트를 아래처럼 수정했다.

```yaml
ports:
  - "127.0.0.1:3000:3000"
```

수정 후 상태는 다음과 같다.

```text
127.0.0.1:3000->3000/tcp
```

이제 Wiki.js는 Raspberry Pi 내부에서만 접근 가능하다.

```text
가능: http://127.0.0.1:3000
불가: 외부에서 집IP:3000 직접 접속
```
<br>

## 9. Cloudflare Zero Trust 설정

Cloudflare에서 **Zero Trust 무료 플랜**을 선택했다.

```text
Zero Trust 무료
$0/사용자/월
```

이후 Cloudflare Tunnel을 생성했다.

```text
Tunnel 이름: junu-wiki
```

Raspberry Pi에는 `cloudflared`를 설치하고 systemd 서비스로 실행했다.

상태 확인 명령어:

```bash
sudo systemctl status cloudflared --no-pager
```

정상 상태:

```text
Active: active (running)
```

---

## 10. Cloudflare Tunnel 상태 확인

Cloudflare Tunnel이 정상적으로 실행 중인지 확인했다.

```bash
sudo systemctl status cloudflared --no-pager
```

로그 확인 명령어:

```bash
sudo journalctl -u cloudflared -n 50 --no-pager
```

정상적으로 Tunnel 연결이 되면 다음과 같은 로그가 보인다.

```text
Registered tunnel connection
Environment is healthy
cloudflared will use 'quic' as primary protocol
```
<br>

## 11. DNS 설정

Cloudflare DNS에는 `wiki.junu.dev`가 Tunnel로 연결되도록 설정되었다.

현재 구조:

```text
wiki.junu.dev
    ↓
Cloudflare Tunnel
    ↓
junu-wiki
```

DNS에서 중요한 점은 다음과 같다.

```text
A 레코드로 집 공인 IP를 직접 바라보지 않음
Tunnel 레코드로 연결
Proxy 상태는 Proxied
```

사용하지 않는 방식:

```text
A    wiki    집 공인 IP
```

현재 사용한 방식:

```text
wiki.junu.dev
    ↓
Tunnel
    ↓
junu-wiki
```
<br>

## 12. Tunnel Route 설정

Cloudflare Tunnel의 Published Application Route를 설정했다.

최종 설정은 다음과 같다.

```text
Hostname: wiki.junu.dev
Path: *
Service: http://127.0.0.1:3000
```

중요한 점은 내부 서비스가 **HTTPS가 아니라 HTTP**라는 것이다.

### 잘못된 설정

```text
https://127.0.0.1:3000
```

이 설정에서는 `502 Bad Gateway` 오류가 발생했다.

로그 내용:

```text
tls: first record does not look like a TLS handshake
originService=https://127.0.0.1:3000
```

### 올바른 설정

```text
http://127.0.0.1:3000
```

외부 접속은 HTTPS지만, Raspberry Pi 내부의 Wiki.js는 HTTP로 연결한다.

```text
브라우저
    ↓
https://wiki.junu.dev
    ↓
Cloudflare
    ↓
Cloudflare Tunnel
    ↓
http://127.0.0.1:3000
    ↓
Wiki.js
```
<br>

## 13. 502 오류 원인과 해결

처음 접속 시 아래 오류가 발생했다.

```text
Bad gateway
Error code 502
```

원인은 Cloudflare Tunnel의 Service URL이 HTTPS로 설정되어 있었기 때문이다.

```text
잘못된 설정: https://127.0.0.1:3000
올바른 설정: http://127.0.0.1:3000
```

Wiki.js의 `3000`번 포트는 HTTPS가 아니라 HTTP로 동작한다.

따라서 Cloudflare Tunnel Route에서 Service를 아래처럼 수정했다.

```text
Service Type: HTTP
Service URL: 127.0.0.1:3000
```

또는 전체 URL 기준으로는 다음과 같다.

```text
http://127.0.0.1:3000
```

수정 후 `wiki.junu.dev` 접속이 정상적으로 동작했다.

---

## 14. Catch-all 404 규칙

Cloudflare Tunnel 화면 아래에 다음 규칙이 보인다.

```text
Catch-all rule: http_status:404
```

이 규칙은 정상이다.

의미는 다음과 같다.

```text
wiki.junu.dev로 들어온 요청 → Wiki.js로 전달
그 외 매칭되지 않는 요청 → 404 반환
```

즉, 불필요한 요청을 막는 fallback 규칙이므로 삭제하지 않아도 된다.

---

## 15. 최종 접속 주소

최종 접속 주소는 다음과 같다.

```text
https://wiki.junu.dev
```

현재 접속 정상 확인 완료.

---

## 16. 최종 구성 요약

```text
Domain
└── junu.dev

Subdomain
└── wiki.junu.dev

Cloudflare
└── Tunnel: junu-wiki
    └── Published Application Route
        └── wiki.junu.dev → http://127.0.0.1:3000

Raspberry Pi
└── cloudflared.service
    └── Wiki.js container
        └── 127.0.0.1:3000

Database
└── wikijs-db
    └── PostgreSQL
```

---

## 17. 자주 사용하는 명령어

### Wiki.js 컨테이너 확인

```bash
docker ps -a --filter "name=wikijs"
```

### Wiki.js 로컬 접속 확인

```bash
curl -I http://127.0.0.1:3000
```

### 외부 도메인 접속 확인

```bash
curl -I https://wiki.junu.dev
```

### Cloudflare Tunnel 상태 확인

```bash
sudo systemctl status cloudflared --no-pager
```

### Cloudflare Tunnel 로그 확인

```bash
sudo journalctl -u cloudflared -n 50 --no-pager
```

### Wiki.js Compose 경로 이동

```bash
cd /home/joon/services/wikijs
```

### Wiki.js 재시작

```bash
docker compose up -d
```

### cloudflared 재시작

```bash
sudo systemctl restart cloudflared
```

---

## 18. 현재 보안 상태

현재 구성은 다음 조건을 만족한다.

```text
공유기 포트포워딩 없음
Wiki.js 3000번 외부 공개 안 함
PostgreSQL 5432번 외부 공개 안 함
Cloudflare Tunnel로만 접근
Wiki.js는 127.0.0.1:3000으로만 바인딩
기존 DB 백업 완료
```

최종적으로 외부에서 직접 열린 포트 없이 `wiki.junu.dev` 접속이 가능해진 상태다.

---

## 19. 주의 사항

### DB 삭제 금지

아래 명령어는 기존 Wiki.js 데이터를 삭제할 수 있으므로 주의해야 한다.

```bash
docker compose down -v
docker volume rm
docker system prune --volumes
docker rm -f wikijs-db
```

### Tunnel Route 수정 시 주의

Wiki.js는 내부에서 HTTP로 동작하므로 Tunnel Route는 반드시 다음처럼 설정해야 한다.

```text
http://127.0.0.1:3000
```

아래처럼 설정하면 안 된다.

```text
https://127.0.0.1:3000
```

### Docker Compose 작업 위치

Wiki.js 관련 Docker Compose 작업은 아래 위치에서 진행해야 한다.

```bash
cd /home/joon/services/wikijs
```

`/srv/junu/wiki`는 백업 파일을 저장한 경로이며, 현재 Wiki.js Compose 프로젝트의 실제 경로가 아니다.

---

## 20. 최종 결론

`wiki.junu.dev`는 Cloudflare Tunnel을 통해 Raspberry Pi의 Wiki.js와 정상 연결되었다.

최종 구조는 다음과 같다.

```text
https://wiki.junu.dev
    ↓
Cloudflare
    ↓
Cloudflare Tunnel
    ↓
Raspberry Pi cloudflared
    ↓
http://127.0.0.1:3000
    ↓
Wiki.js
```

이 방식은 공유기 포트포워딩을 사용하지 않기 때문에 개인 위키 서버 운영 방식으로 비교적 안전하다.