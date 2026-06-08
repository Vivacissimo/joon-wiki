---
title: PostgreSQL로 DB 백업
description: 
published: true
date: 2026-06-08T07:43:00.112Z
tags: 
editor: markdown
dateCreated: 2026-06-04T07:31:24.628Z
---

# Wiki.js PostgreSQL 백업 및 복구 방법

## 1. 목적

Wiki.js의 데이터베이스를 PostgreSQL dump 파일로 백업하여 라즈베리파이 장애, Docker Volume 삭제, SD카드 손상 등의 상황에서도 Wiki.js를 복구할 수 있게 구성한다.

GitHub 연동은 주로 문서 본문을 백업하는 용도이고, PostgreSQL 백업은 Wiki.js 전체 복구에 필요하다.

```txt
GitHub 연동
→ Markdown 문서 백업

PostgreSQL 백업
→ Wiki.js 전체 데이터 백업
→ 사용자 계정, 권한, 페이지 정보, 설정 데이터 보존
```

---

## 2. 현재 구성 확인

Wiki.js 작업 폴더로 이동한다.

```bash
cd ~/services/wikijs
```

컨테이너 상태를 확인한다.

```bash
docker compose ps
```

정상이라면 아래 컨테이너가 실행 중이어야 한다.

```txt
wikijs
wikijs-db
```

현재 `.env` 파일을 확인한다.

```bash
cat .env
```

예시:

```env
POSTGRES_DB=wiki
POSTGRES_USER=wikijs
POSTGRES_PASSWORD=ChangeThisPassword_1234!
```

현재 구성 기준:

```txt
DB Container: wikijs-db
Database: wiki
DB User: wikijs
DB Volume: db_data
```

---

## 3. 백업 폴더 생성

라즈베리파이에 백업 파일을 저장할 폴더를 만든다.

```bash
mkdir -p ~/backup/wikijs/postgres
```

확인:

```bash
ls -ld ~/backup/wikijs/postgres
```

---

## 4. 수동 백업

Wiki.js 작업 폴더에서 실행한다.

```bash
cd ~/services/wikijs
```

PostgreSQL DB를 SQL 파일로 백업한다.

```bash
docker compose exec -T db pg_dump -U wikijs wiki > ~/backup/wikijs/postgres/wiki_$(date +%Y%m%d_%H%M%S).sql
```

백업 파일 확인:

```bash
ls -lh ~/backup/wikijs/postgres
```

예시 결과:

```txt
wiki_20260604_153000.sql
```

---

## 5. 압축 백업

SQL 파일을 바로 gzip으로 압축해서 저장할 수도 있다.

```bash
cd ~/services/wikijs
docker compose exec -T db pg_dump -U wikijs wiki | gzip > ~/backup/wikijs/postgres/wiki_$(date +%Y%m%d_%H%M%S).sql.gz
```

압축 백업 파일 확인:

```bash
ls -lh ~/backup/wikijs/postgres
```

압축 파일 내용 확인:

```bash
gzip -l ~/backup/wikijs/postgres/*.sql.gz
```

---

## 6. 백업 파일을 내 PC로 복사

라즈베리파이에만 백업을 두면 라즈베리파이 저장장치가 고장 났을 때 같이 날아갈 수 있다.

따라서 중요한 백업 파일은 PC, NAS, 외장 SSD, Google Drive, S3 등에 한 번 더 복사하는 것이 좋다.

Mac 또는 Linux PC에서 실행:

```bash
scp joon@192.168.35.200:/home/joon/backup/wikijs/postgres/wiki_*.sql.gz .
```

특정 파일만 복사:

```bash
scp joon@192.168.35.200:/home/joon/backup/wikijs/postgres/wiki_20260604_153000.sql.gz .
```

---

## 7. 자동 백업 설정

매일 자동으로 PostgreSQL 백업을 남기도록 cron을 설정한다.

```bash
crontab -e
```

아래 내용을 추가한다.

```cron
0 3 * * * cd /home/joon/services/wikijs && docker compose exec -T db pg_dump -U wikijs wiki | gzip > /home/joon/backup/wikijs/postgres/wiki_$(date +\%Y\%m\%d_\%H\%M\%S).sql.gz
```


의미:

```txt
매일 새벽 3시에 Wiki.js PostgreSQL DB를 백업한다.
백업 파일은 /home/joon/backup/wikijs/postgres 아래에 저장한다.
파일명에는 날짜와 시간이 붙는다.
```

cron 등록 확인:

```bash
crontab -l
```

---

## 8. 오래된 백업 자동 삭제

백업 파일이 계속 쌓이면 저장공간을 많이 사용할 수 있다.

30일이 지난 백업 파일을 자동 삭제하려면 cron에 아래 줄도 추가한다.

```cron
30 3 * * * find /home/joon/backup/wikijs/postgres -name "wiki_*.sql.gz" -mtime +30 -delete
```

의미:

```txt
매일 새벽 3시 30분에 30일이 지난 백업 파일을 삭제한다.
```

---

## 9. 백업 스크립트로 관리

명령어가 길기 때문에 백업 스크립트로 만들어두는 것이 좋다.

스크립트 폴더 생성:

```bash
mkdir -p ~/scripts
```

백업 스크립트 생성:

```bash
nano ~/scripts/backup_wikijs_postgres.sh
```

내용 입력:

```bash
#!/bin/bash

set -e

BACKUP_DIR="/home/joon/backup/wikijs/postgres"
WIKI_DIR="/home/joon/services/wikijs"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

cd "$WIKI_DIR"

docker compose exec -T db pg_dump -U wikijs wiki | gzip > "$BACKUP_DIR/wiki_${DATE}.sql.gz"

find "$BACKUP_DIR" -name "wiki_*.sql.gz" -mtime +30 -delete

echo "Wiki.js PostgreSQL backup completed: $BACKUP_DIR/wiki_${DATE}.sql.gz"
```

실행 권한 부여:

```bash
chmod +x ~/scripts/backup_wikijs_postgres.sh
```

수동 실행 테스트:

```bash
~/scripts/backup_wikijs_postgres.sh
```

백업 파일 확인:

```bash
ls -lh ~/backup/wikijs/postgres
```

cron에는 스크립트만 등록한다.

```bash
crontab -e
```

아래 내용 추가:

```cron
0 3 * * * /home/joon/scripts/backup_wikijs_postgres.sh >> /home/joon/backup/wikijs/postgres/backup.log 2>&1
```

로그 확인:

```bash
tail -f ~/backup/wikijs/postgres/backup.log
```

---

## 10. 복구 전 주의사항

복구는 기존 DB 데이터를 덮어쓰는 작업이다.

복구 전에 반드시 현재 상태도 한 번 백업한다.

```bash
cd ~/services/wikijs
docker compose exec -T db pg_dump -U wikijs wiki | gzip > ~/backup/wikijs/postgres/wiki_before_restore_$(date +%Y%m%d_%H%M%S).sql.gz
```

그리고 Wiki.js 컨테이너를 잠시 중지한다.

```bash
cd ~/services/wikijs
docker compose stop wiki
```

DB 컨테이너는 살아 있어야 한다.

```bash
docker compose ps
```

---

## 11. 압축 백업 파일 복구

복구할 백업 파일을 확인한다.

```bash
ls -lh ~/backup/wikijs/postgres
```

예시 백업 파일:

```txt
wiki_20260604_153000.sql.gz
```

기존 DB를 초기화한 뒤 복구한다.

```bash
cd ~/services/wikijs
```

기존 DB 삭제:

```bash
docker compose exec -T db dropdb -U wikijs wiki
```

새 DB 생성:

```bash
docker compose exec -T db createdb -U wikijs wiki
```

압축 백업 파일 복구:

```bash
gunzip -c ~/backup/wikijs/postgres/wiki_20260604_153000.sql.gz | docker compose exec -T db psql -U wikijs wiki
```

Wiki.js 다시 시작:

```bash
docker compose start wiki
```

상태 확인:

```bash
docker compose ps
```

브라우저에서 접속 확인:

```txt
http://192.168.35.200:3000
```

---

## 12. SQL 파일 복구

압축하지 않은 `.sql` 파일을 복구할 때는 아래처럼 실행한다.

```bash
cd ~/services/wikijs
docker compose stop wiki
docker compose exec -T db dropdb -U wikijs wiki
docker compose exec -T db createdb -U wikijs wiki
docker compose exec -T db psql -U wikijs wiki < ~/backup/wikijs/postgres/wiki_20260604_153000.sql
docker compose start wiki
```

---

## 13. 전체 Docker Volume 백업 방식

PostgreSQL은 SQL dump 방식이 가장 안전하고 이식성이 좋다.

다만 Docker Volume 자체를 백업하고 싶다면 아래 방식도 가능하다.

현재 볼륨 이름 확인:

```bash
docker volume ls
```

대략 아래와 같은 볼륨이 보일 수 있다.

```txt
wikijs_db_data
```

볼륨 백업 예시:

```bash
docker run --rm \
  -v wikijs_db_data:/volume \
  -v /home/joon/backup/wikijs/postgres:/backup \
  alpine \
  tar czf /backup/wikijs_db_data_$(date +%Y%m%d_%H%M%S).tar.gz -C /volume .
```

단, PostgreSQL은 실행 중인 데이터 파일을 그대로 복사하면 일관성 문제가 생길 수 있으므로 일반 백업은 `pg_dump` 방식을 우선 사용한다.

---

## 14. 백업 테스트 체크리스트

백업은 만들어두는 것보다 복구 테스트가 더 중요하다.

정기적으로 아래를 확인한다.

```txt
1. 백업 파일이 매일 생성되는가?
2. 백업 파일 크기가 0 byte가 아닌가?
3. gzip 압축 파일이 정상인가?
4. 별도 PC나 외부 저장소에 복사되고 있는가?
5. 테스트 환경에서 복구가 되는가?
```

확인 명령어:

```bash
ls -lh ~/backup/wikijs/postgres
gzip -t ~/backup/wikijs/postgres/wiki_파일명.sql.gz
echo $?
```

`echo $?` 결과가 `0`이면 gzip 파일이 정상이다.

---

## 15. 최종 운영 기준

권장 운영 방식:

```txt
매일 새벽 3시 PostgreSQL 자동 백업
30일 지난 백업 자동 삭제
중요 백업은 PC/NAS/클라우드에 추가 복사
Wiki.js 업데이트 전 수동 백업
라즈베리파이 OS 재설치 전 수동 백업
월 1회 복구 테스트
```

---

## 16. 정리

```txt
PostgreSQL 백업은 Wiki.js 전체 복구를 위한 핵심 백업이다.
GitHub 연동은 문서 백업용이고, DB 백업을 대체하지 않는다.
pg_dump로 SQL 백업을 생성한다.
복구 시에는 Wiki.js 컨테이너를 중지하고 DB를 복원한다.
백업 파일은 라즈베리파이 외부에도 반드시 복사한다.
```