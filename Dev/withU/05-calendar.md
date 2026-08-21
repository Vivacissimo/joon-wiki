---
title: withU 일정 및 Apple Calendar
description: 커플 일정 공유와 EventKit 연동 구조
published: true
date: 2026-08-21T07:39:00.000Z
tags: withu,calendar,eventkit,ios
editor: markdown
dateCreated: 2026-08-21T07:39:00.000Z
---

# withU 일정 및 Apple Calendar

## 1. 목표

커플이 서로의 일정을 앱 안에서 공유하고, 사용자가 원할 경우 iPhone 기본 캘린더에 있는 일정을 선택해서 withU로 가져올 수 있게 한다.

## 2. 일정 등록 방식

일정은 두 경로로 생성된다.

```text
1. withU에서 직접 일정 생성
2. Apple Calendar에서 선택해 가져오기
```

모든 공유 일정은 `CoupleSchedule`에 저장되고, 같은 `coupleId`를 가진 두 사용자가 함께 볼 수 있다.

## 3. Apple Calendar 가져오기

```text
iPhone Calendar
      ↓
   EventKit
      ↓
사용자가 공유할 일정 선택
      ↓
Backend API
      ↓
CoupleSchedule 저장
      ↓
상대방과 공유
```

개인 캘린더 전체를 자동으로 상대방에게 노출하지 않는다.

사용자가 직접 선택한 일정만 공유한다.

## 4. withU 일정을 Apple Calendar에 추가

```text
CoupleSchedule
      ↓
[내 iPhone 캘린더에 추가]
      ↓
EventKit
      ↓
Apple Calendar
```

1차 MVP에서는 자동 동기화가 아니라 사용자가 버튼을 눌러 추가하는 방식으로 구현한다.

## 5. 캘린더 화면

하나의 월간 캘린더에서 일정과 추억을 함께 보여주는 방향을 목표로 한다.

```text
📷 작성된 다이어리
📅 예정된 일정
❤️ 기념일
```

날짜를 선택하면 해당 날짜의 일정과 다이어리를 함께 볼 수 있게 한다.

## 6. MVP에서 제외

- Apple Calendar 변경사항 실시간 감지
- withU ↔ Apple Calendar 자동 양방향 동기화
- 상대방의 개인 캘린더 직접 조회
- Google Calendar 연동

## 7. iOS 구조

```text
Services
├── ScheduleService
└── CalendarService
      └── EventKit
```

`CalendarService`는 기기 캘린더 접근을 담당하고, `ScheduleService`는 서버의 커플 공유 일정 CRUD를 담당한다.
