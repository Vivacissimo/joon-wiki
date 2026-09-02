---
title: 04. Pod
description: Pod 기본 개념
published: true
tags: kubernetes
editor: markdown
---

# Pod

Kubernetes에서 컨테이너를 실행하는 **가장 작은 배포 단위**.

컨테이너를 Kubernetes에 직접 배포하는 것이 아니라 Pod 안에 넣어 실행.

```text
Kubernetes
└── Pod
    └── Container
```

## Pod와 Container

대부분의 경우 Pod 하나에 Application Container 하나를 실행.

```text
Pod
└── nginx container
```

필요하면 하나의 Pod 안에 여러 Container도 구성 가능.

```text
Pod
├── Application Container
└── Sidecar Container
```

같은 Pod의 Container는 다음을 공유.

- Pod IP
- Network Namespace
- Volume
- Lifecycle

## Pod IP

각 Pod는 Cluster 내부에서 사용할 IP를 하나씩 받음.

```text
Pod A → 192.168.1.10
Pod B → 192.168.2.15
```

Pod는 재생성될 수 있기 때문에 IP도 바뀔 수 있음.

따라서 Application 접근은 보통 Pod IP가 아니라 `Service` 사용.

## Pod 생성

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
```

## 상태 확인

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
kubectl logs nginx
```

## 주요 상태

| 상태 | 의미 |
|---|---|
| Pending | 아직 실행 위치 결정 또는 Image 준비 중 |
| Running | 정상 실행 중 |
| Succeeded | 작업 완료 |
| Failed | 실행 실패 |
| Unknown | Node와 상태 통신 불가 |

## Pod를 직접 운영하지 않는 이유

Pod 자체에는 복제 수 유지나 자동 교체 기능이 부족.

운영 환경에서는 대부분 Deployment 같은 Controller 사용.

```text
Deployment
└── ReplicaSet
    ├── Pod
    ├── Pod
    └── Pod
```

## 다음

`05-Workload`에서 Deployment, ReplicaSet, StatefulSet, DaemonSet 정리.
