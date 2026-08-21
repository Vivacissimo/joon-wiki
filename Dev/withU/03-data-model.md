---
title: withU 데이터 모델
description: 1차 MVP 기준 주요 엔티티와 관계
published: true
date: 2026-08-21T07:39:00.000Z
tags: withu,database,data-model
editor: markdown
dateCreated: 2026-08-21T07:39:00.000Z
---

# withU 데이터 모델

## 1. 핵심 엔티티

```text
User
 └── Couple
      ├── Diary
      │    ├── DiaryImage
      │    ├── DiaryReaction
      │    └── Comment
      └── CoupleSchedule
```

## 2. User

```text
User
- id
- appleUserId
- nickname
- profileImageUrl
- coupleId
- createdAt
```

## 3. Couple

```text
Couple
- id
- user1Id
- user2Id
- anniversary
- createdAt
```

두 사용자가 같은 `coupleId`를 기준으로 다이어리와 일정을 공유한다.

## 4. Diary

```text
Diary
- id
- coupleId
- authorId
- title
- content
- diaryDate
- mood
- createdAt
- updatedAt
```

## 5. DiaryImage

```text
DiaryImage
- id
- diaryId
- imageKey
- order
- createdAt
```

이미지 바이너리는 DB에 넣지 않고 S3에 저장한다. DB에는 S3 객체의 `imageKey`만 저장한다.

## 6. DiaryReaction

```text
DiaryReaction
- id
- diaryId
- userId
- reactionType
```

1차 MVP에서는 한 사용자가 하나의 다이어리에 하나의 반응만 갖도록 단순화할 수 있다.

예시 타입:

```text
LOVE
HEART
LAUGH
SAD
LIKE
```

## 7. Comment

```text
Comment
- id
- diaryId
- authorId
- content
- createdAt
- updatedAt
```

댓글 반응은 2차 기능으로 분리한다.

## 8. CoupleSchedule

```text
CoupleSchedule
- id
- coupleId
- creatorId
- title
- description
- location
- startDate
- endDate
- isAllDay
- source
- createdAt
- updatedAt
```

`source` 예시:

```text
APP
APPLE_CALENDAR
```

## 9. 권한 원칙

모든 커플 전용 데이터 조회 시 로그인 사용자의 `coupleId`와 리소스의 `coupleId`가 일치하는지 검증한다.

```text
resource.coupleId == currentUser.coupleId
```

이 검증은 클라이언트가 아니라 반드시 백엔드에서도 수행한다.
