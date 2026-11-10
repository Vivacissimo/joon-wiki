---
title: SAA 05 — 스토리지
description: S3, EBS, EFS, FSx와 보관 계층·백업·접근 제어 비교
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 스토리지

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 데이터 접근 방식부터 고른다

| 유형 | 서비스 | 선택 단서 |
|---|---|---|
| 객체 | S3 | 이미지·백업·로그·데이터 레이크, API 접근 |
| 블록 | EBS | EC2 OS·파일 시스템·DB 볼륨 |
| 임시 블록 | Instance Store | 재생성 가능한 캐시·중간 결과 |
| 공유 파일 | EFS | Linux NFS 공유 |
| 관리형 파일 시스템 | FSx | Windows SMB, Lustre 등 특정 파일 시스템 |

S3는 일반적인 POSIX 공유 디스크가 아니다. Instance Store는 중지·종료 등의 이벤트에서 데이터가 사라질 수 있으므로 유일한 영구 저장소로 사용하지 않는다. EBS는 인스턴스 종료 시 삭제 설정을 확인한다. [EC2 스토리지 비교](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html)

## 2. EBS·EFS·FSx

| 서비스 | 범위·특성 | 주의 |
|---|---|---|
| EBS | AZ 단위 볼륨, 스냅샷으로 복구·이전 | 다른 AZ 인스턴스에 그대로 연결하지 못함 |
| EFS Regional | 여러 AZ에서 접근하는 공유 파일 | 마운트 대상·SG·성능 모드 확인 |
| EFS One Zone | 단일 AZ 파일 시스템 | AZ 장애 요구와 비교 |
| FSx for Windows File Server | SMB·Windows 파일 공유 | AD 통합 등 Windows 요구 |
| FSx for Lustre | 고성능 병렬 파일 처리 | HPC·분석 등 워크로드 요구 |

일부 EBS 유형의 Multi-Attach는 조건부 기능이다. 일반 볼륨을 여러 AZ의 서버가 동시에 공유하는 대체재로 외우지 않는다. [EBS Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html), [EFS 구성](https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html)

### EBS 성능 선택

`gp3`는 일반적인 SSD 워크로드의 출발점이다. 높은 일관된 IOPS·낮은 지연이 중요하면 Provisioned IOPS SSD 계열을 비교한다. 처리량 중심의 순차 접근과 IOPS 중심의 랜덤 접근을 구분한다. 용량을 늘리는 것이 모든 성능 병목의 답은 아니다. [EBS 볼륨 유형](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)

## 3. S3 보관 계층

| 계층 | 적합한 조건 | 주의 |
|---|---|---|
| Standard | 자주 접근 | 저장량·요청 비용 |
| Intelligent-Tiering | 접근 패턴이 변하거나 불명확 | 모니터링 비용, 선택한 아카이브 계층의 복원 특성 |
| Standard-IA | 드물게 읽지만 즉시 필요 | 검색 요금·최소 보관 기간 |
| One Zone-IA | 재생성 가능한 드문 접근 데이터 | 단일 AZ |
| Glacier Instant Retrieval | 드문 접근, 밀리초 조회 필요 | 검색 요금·최소 보관 기간 |
| Glacier Flexible Retrieval | 복원 대기 허용 | 즉시 GET하는 계층으로 가정 금지 |
| Glacier Deep Archive | 장기 보관, 긴 복원 대기 허용 | 복원 시간·최소 보관 기간 |
| Express One Zone | 단일 AZ의 낮은 지연 객체 접근 | 다중 AZ 내구성 요구와 비교 |

표는 선택 방향이다. 가장 낮은 GB당 가격만 비교하지 않는다. 객체 크기·요청량·검색량·최소 보관 기간·복원 SLA를 함께 확인한다. [스토리지 클래스 공식 비교](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)

## 4. S3 기능을 요구사항에 연결하기

| 요구사항 | 기능 |
|---|---|
| 덮어쓰기·실수 삭제에 대비 | Versioning |
| 정해진 기간 뒤 계층 전환·만료 | Lifecycle |
| 다른 버킷·리전으로 객체 복제 | Replication |
| 규정 기간 동안 변경·삭제 방지 | Object Lock |
| 제한된 시간 동안 객체 접근 위임 | Presigned URL |
| 대용량 객체를 부분 단위 업로드 | Multipart Upload |
| 먼 거리의 S3 업로드 전송 개선 | Transfer Acceleration 검토 |

복제는 모든 사용자 실수를 원상 복구하는 독립 백업과 같지 않다. 버전·삭제 마커·기존 객체 복제 조건을 확인한다. Versioning에서는 삭제 마커와 실제 버전 영구 삭제를 구분한다. S3 객체 쓰기·덮어쓰기·삭제와 목록 조회는 강한 일관성을 제공하며, CloudFront 같은 외부 캐시의 오래된 응답은 별도 문제이다. [S3 기능과 일관성](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

## 5. S3 보안

버킷을 공개하지 않고 CloudFront로 배포하려면 일반 S3 버킷 오리진에 OAC와 적절한 버킷 정책을 사용한다. S3 웹사이트 엔드포인트는 일반 버킷 오리진과 인증 방식이 다르다. Block Public Access와 최소 권한도 함께 확인한다. [OAC로 S3 접근 제한](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)

SSE-S3·SSE-KMS는 서버 측 암호화 방식이다. 고객 관리 KMS 키의 권한 제어가 필요하면 SSE-KMS를 검토한다. HTTPS는 전송 중 암호화로 저장 시 암호화와 별개이다. [S3 암호화](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)

다음: [데이터베이스와 캐시](/en/Study/AWS/Certification/SAA/06-Databases)
