---
title: withU 이미지 스토리지
description: 사진 업로드 및 S3 저장 구조
published: true
date: 2026-08-21T07:39:00.000Z
tags: withu,s3,image,storage
editor: markdown
dateCreated: 2026-08-21T07:39:00.000Z
---

# withU 이미지 스토리지

## 1. 기본 원칙

사진 파일 자체는 DB에 저장하지 않고 **오브젝트 스토리지(S3)** 에 저장한다.

DB에는 사진이 어느 다이어리에 속하는지와 S3 객체 키만 저장한다.

```text
iOS App
   │
   │ 업로드 URL 요청
   ▼
Backend API
   │
   │ Presigned URL 발급
   ▼
iOS App ───────────────► S3
        사진 직접 업로드

Backend
   │
   ▼
Database
- diaryId
- imageKey
- order
```

## 2. S3 경로 예시

```text
couples/
└── couple_abc/
    └── diaries/
        └── diary_001/
            ├── 01_uuid.jpg
            ├── 02_uuid.jpg
            └── 03_uuid.jpg
```

## 3. 업로드 흐름

1. iOS에서 `PhotosPicker`로 사진 선택
2. 백엔드에 업로드 요청
3. 백엔드가 사용자 및 `coupleId` 확인
4. S3용 Presigned PUT URL 생성
5. iOS가 S3에 직접 업로드
6. 업로드 성공 후 `imageKey`를 백엔드에 전달
7. `DiaryImage`에 저장

## 4. 보안

S3 버킷은 **Private**으로 유지한다.

앱 내부에 AWS Access Key / Secret Key를 넣지 않는다.

조회 시에도 필요하면 백엔드에서 Presigned GET URL을 생성해서 클라이언트에 전달한다.

## 5. MVP 범위

1차에서는 아래까지만 구현한다.

- 사진 여러 장 선택
- 업로드 전 적당한 압축
- S3 직접 업로드
- Private 버킷
- 사진 순서 저장

아래는 2차 이후로 미룬다.

- Lambda 기반 자동 리사이징
- 썸네일/medium/original 다중 버전
- CDN
- AI 이미지 분석
