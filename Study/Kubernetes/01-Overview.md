---
title: 01. Overview
description: Kubernetes 기본 개념
published: true
tags: kubernetes
editor: markdown
---

# Kubernetes

컨테이너를 여러 서버에서 자동으로 배치하고 운영하기 위한 컨테이너 오케스트레이션 플랫폼.

## 왜 필요한가

컨테이너 수가 늘어나면 직접 관리하기 어려운 부분이 생김.

- 어느 서버에 컨테이너를 실행할지 결정
- 장애 난 컨테이너 재시작
- 여러 컨테이너에 트래픽 분산
- 애플리케이션 확장/축소
- 버전 업데이트와 롤백
- 설정값과 Secret 관리

Kubernetes가 이런 작업을 선언형 방식으로 자동화.

## 기본 흐름

```text
Application
    ↓
Container Image
    ↓
Pod
    ↓
Deployment
    ↓
Service
    ↓
Kubernetes Cluster
```

## 핵심 특징

| 기능 | 의미 |
|---|---|
| Scheduling | Pod를 실행할 Node 선택 |
| Self-healing | 장애 난 Pod 자동 복구 |
| Scaling | Pod 개수 확장/축소 |
| Service Discovery | Service 이름으로 Pod 접근 |
| Load Balancing | 여러 Pod로 트래픽 분산 |
| Rolling Update | 서비스 중단 최소화하며 버전 교체 |
| Declarative Management | 원하는 상태를 YAML로 정의 |

## Docker와 차이

Docker는 컨테이너를 만드는 기술.

Kubernetes는 만들어진 컨테이너를 여러 서버에서 운영하는 기술.

```text
Docker
→ Image / Container

Kubernetes
→ Pod / Deployment / Service / Cluster
```

## 다음

`02-Architecture`에서 Kubernetes 전체 구조와 Control Plane / Worker Node 구성 확인.
