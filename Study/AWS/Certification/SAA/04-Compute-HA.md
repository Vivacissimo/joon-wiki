---
title: SAA 04 — 컴퓨팅과 고가용성
description: EC2, 컨테이너, Lambda와 로드 밸런싱·자동 확장 설계
published: true
date: 2026-11-10T00:00:00.000Z
tags: aws, certification, saa
editor: markdown
dateCreated: 2026-11-10T00:00:00.000Z
---

# 컴퓨팅과 고가용성

[SAA 목차](/en/Study/AWS/Certification/SAA)

## 1. 실행 환경 선택

| 환경 | 적합한 조건 | 확인할 제약 |
|---|---|---|
| EC2 | OS·에이전트·런타임 제어 필요 | 패치·용량·장애 대응 책임 |
| ECS | AWS 통합 컨테이너 오케스트레이션 | 실행 용량을 EC2 또는 Fargate 등으로 선택 |
| EKS | Kubernetes API·생태계 필요 | 클러스터·애플리케이션 운영 복잡도 |
| Fargate | 서버 관리 부담 없이 컨테이너 실행 | 지원하는 리소스·네트워크·워크로드 제약 |
| Lambda | 이벤트 기반 함수 실행 | 실행 방식별 시간·동시성·런타임 제약 |
| AWS Batch | 배치 작업의 대기열·스케줄링 | 작업 특성에 맞는 컴퓨팅 환경 |

ECS·EKS는 오케스트레이션, Fargate는 컨테이너 실행 용량의 선택이다. ECR은 컨테이너 이미지 저장소이다. [ECS 공식 개요](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)

일반적인 Lambda 함수 호출은 최대 15분을 기준으로 판단한다. 현재 공식 문서에는 특정 Lambda Managed Instances 실행 방식의 예외가 있으므로 **모든 Lambda 실행이 무조건 15분 제한**이라고 일반화하지 않는다. 문제의 실행 모델을 확인한다. [Lambda 할당량](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)

## 2. ALB·NLB·GWLB

| 서비스 | 주 용도 | 문제에서 볼 단서 |
|---|---|---|
| ALB | HTTP·HTTPS 애플리케이션 분산 | 호스트·경로 기반 라우팅 |
| NLB | 전송 계층 트래픽 분산 | TCP·UDP, 고정 IP 요구 |
| GWLB | 네트워크 어플라이언스 트래픽 분산 | 방화벽·검사 장비 삽입 |

로드 밸런서는 정상 대상으로 요청을 보낸다. ASG는 인스턴스 수를 유지·조절한다. **대상을 제외하는 것과 새 인스턴스로 교체하는 것은 별개**이다. ASG에서 로드 밸런서 상태 검사를 활용하도록 설정했는지도 본다. [ELB 개요](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html), [상태 검사](https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-checks-overview.html)

## 3. 확장과 고가용성

| 개념 | 의미 | 한계 |
|---|---|---|
| 수직 확장 | 인스턴스를 더 큰 사양으로 변경 | 단일 자원 한계와 변경 중단 가능성 |
| 수평 확장 | 인스턴스 수 증가 | 공유 상태와 동시 처리 설계 필요 |
| 다중 AZ | AZ 장애에 대비 | 리전 전체 장애까지 자동 해결하지 않음 |
| 다중 리전 | 리전 장애·글로벌 요구 대응 | 데이터 복제·일관성·비용 복잡도 |

CPU가 높은 경우 앱을 늘리는 것과 DB 병목을 해결하는 것은 다르다. 병목 지표가 CPU인지, 큐 적체인지, 연결 수인지, DB 읽기인지 먼저 본다.

## 4. ASG 정책 선택

| 정책 | 적합한 상황 |
|---|---|
| Target Tracking | CPU 등 특정 지표의 목표값 유지 |
| Step Scaling | 알람 위반 정도에 따라 다른 규모로 확장 |
| Scheduled Scaling | 매일 정해진 시간의 예측 가능한 수요 |
| Predictive Scaling | 반복적인 과거 수요 패턴을 예측 |

확장은 즉시 끝나지 않는다. 부팅·초기화·헬스 체크·워밍업 시간이 있다. 시작 시각이 확실한 행사라면 사전 확장도 비교한다. [ASG 확장 방법](https://docs.aws.amazon.com/autoscaling/ec2/userguide/scale-your-group.html)

## 5. 상태를 외부로 분리하기

```text
ALB → 앱 A / 앱 B / 앱 C
          ↓          ↓
      공유 세션 캐시   S3·EFS 등 공유 데이터
```

로컬 디스크에 세션·업로드를 저장하면 요청이 다른 인스턴스로 갔을 때 데이터가 보이지 않을 수 있다. 세션은 공유 저장소로, 파일은 접근 방식에 맞는 공유 스토리지로 분리한다. Sticky Session은 특정 대상으로 묶는 기능이며 대상 장애 시 세션 내구성을 보장하는 저장소가 아니다.

## 6. EC2 특수 요구

| 요구 | 검토할 것 |
|---|---|
| 같은 AZ에서 노드 간 낮은 지연 | Cluster Placement Group |
| 소수 중요 인스턴스의 하드웨어 분리 | Spread Placement Group |
| 분산 시스템의 파티션별 장애 격리 | Partition Placement Group |
| CPU 중심 계산 | Compute Optimized 계열 |
| 큰 인메모리 데이터셋 | Memory Optimized 계열 |
| 중단 가능 계산 비용 절감 | Spot + 재시도·체크포인트 |

배치 그룹의 성능과 장애 격리 목적은 다르다. Cluster 배치 그룹만으로 다중 AZ 고가용성을 얻는 것은 아니다. [배치 그룹](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)

다음: [스토리지](/en/Study/AWS/Certification/SAA/05-Storage)
