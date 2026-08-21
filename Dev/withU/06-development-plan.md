---
title: withU 개발 계획
description: 1차 MVP 구현 순서와 마일스톤
published: true
date: 2026-08-21T07:39:00.000Z
tags: withu,development,roadmap
editor: markdown
dateCreated: 2026-08-21T07:39:00.000Z
---

# withU 개발 계획

## 1. 구현 우선순위

```text
인증 / User / Couple
      ↓
Diary CRUD
      ↓
PhotosPicker + S3 업로드
      ↓
댓글 + 다이어리 Reaction
      ↓
CoupleSchedule
      ↓
EventKit / Apple Calendar
      ↓
UI 정리 + 버그 수정
      ↓
TestFlight
```

## 2. 단계별 목표

### 1단계 — 계정 및 커플 연결

- Sign in with Apple
- 사용자 생성
- 닉네임 / 프로필
- 초대코드 생성
- 커플 연결 및 해제

### 2단계 — 다이어리

- 다이어리 생성 / 조회 / 수정 / 삭제
- 피드
- 상세 화면
- 작성자 및 날짜 표시
- 기분 이모티콘

### 3단계 — 사진

- `PhotosPicker`
- 이미지 압축
- Presigned URL API
- S3 업로드
- `DiaryImage` 저장

### 4단계 — 소통

- 댓글 작성 / 삭제
- 다이어리 이모티콘 반응
- 사용자당 하나의 반응을 기본 정책으로 적용

### 5단계 — 커플 일정

- 일정 생성 / 수정 / 삭제
- 월간 캘린더
- 상대방과 일정 공유
- 홈에 오늘 일정 노출

### 6단계 — Apple Calendar

- EventKit 권한 처리
- 기기 캘린더 일정 조회
- 사용자가 선택한 일정만 withU에 복사
- withU 일정을 iPhone 캘린더에 추가

### 7단계 — 안정화

- 빈 화면 / 오류 상태 처리
- 이미지 업로드 실패 처리
- 권한 거부 UX
- 커플 해제 시 데이터 접근 검증
- 실기기 2대 테스트
- TestFlight 배포

## 3. 1차 완료 기준

1차 MVP는 아래 시나리오가 모두 성공하면 완료로 본다.

1. A와 B가 각각 로그인한다.
2. 초대코드로 커플 연결한다.
3. A가 사진이 포함된 다이어리를 작성한다.
4. B의 기기에서 해당 글과 사진이 보인다.
5. B가 댓글과 반응을 남긴다.
6. A가 공유 일정을 등록하면 B에게도 보인다.
7. 사용자가 자신의 Apple Calendar에서 선택한 일정을 withU로 가져올 수 있다.
8. withU 일정을 iPhone 기본 캘린더에 추가할 수 있다.
