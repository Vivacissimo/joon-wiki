---
title: 05. Workload
description: Kubernetes Workload 개념
published: true
tags: kubernetes
editor: markdown
---

# Workload

Pod를 어떤 방식으로 생성하고 유지할지 관리하는 리소스.

## Deployment

일반적인 웹 애플리케이션이나 API 서버에 가장 많이 사용.

주요 기능:

- Pod 개수 유지
- Rolling Update
- Rollback
- ReplicaSet 자동 관리

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

```text
Deployment
└── ReplicaSet
    ├── Pod
    ├── Pod
    └── Pod
```

## ReplicaSet

지정한 개수의 Pod가 항상 유지되도록 관리.

보통 직접 생성하지 않고 Deployment가 자동 관리.

## StatefulSet

Pod마다 고정된 이름과 Storage가 필요한 경우 사용.

예시:

- Database
- Kafka
- Elasticsearch

```text
mysql-0
mysql-1
mysql-2
```

## DaemonSet

각 Node마다 Pod 하나씩 실행할 때 사용.

예시:

- 로그 수집 Agent
- Monitoring Agent
- CNI Agent

```text
Node 1 → Agent Pod
Node 2 → Agent Pod
Node 3 → Agent Pod
```

## Job

한 번 실행하고 종료되는 작업.

예시:

- Batch
- DB Migration
- 일회성 데이터 처리

## CronJob

정해진 시간마다 Job 실행.

예시:

- 매일 백업
- 주기적 정리 작업

## 선택 기준

| 목적 | 리소스 |
|---|---|
| 일반 Application | Deployment |
| 상태가 필요한 Application | StatefulSet |
| 모든 Node에 실행 | DaemonSet |
| 일회성 작업 | Job |
| 주기적 작업 | CronJob |

## 다음

`06-Service-and-Network`에서 Pod에 안정적으로 접근하는 방법 확인.
