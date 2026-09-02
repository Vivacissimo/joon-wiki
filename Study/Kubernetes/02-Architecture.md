---
title: 02. Architecture
description: Kubernetes 아키텍처
published: true
tags: kubernetes
editor: markdown
---

# Kubernetes Architecture

Kubernetes Cluster는 크게 **Control Plane**과 **Worker Node**로 구분.

```text
Kubernetes Cluster
├── Control Plane
│   ├── kube-apiserver
│   ├── etcd
│   ├── kube-scheduler
│   └── kube-controller-manager
│
└── Worker Node
    ├── kubelet
    ├── container runtime
    ├── kube-proxy
    └── Pod
```

## Control Plane

클러스터 전체 상태 관리.

### kube-apiserver

Kubernetes의 진입점.

`kubectl`, Controller, kubelet 등이 API Server를 통해 통신.

```text
kubectl
   ↓
kube-apiserver
   ↓
Kubernetes Resources
```

### etcd

클러스터 상태를 저장하는 Key-Value DB.

저장 대상 예시:

- Node 정보
- Pod 상태
- Deployment 설정
- Secret / ConfigMap
- Service 정보

### kube-scheduler

새 Pod를 어느 Worker Node에 배치할지 결정.

CPU, Memory, Taint/Toleration, Affinity 등을 참고.

### kube-controller-manager

현재 상태를 원하는 상태로 맞추는 Controller 실행.

예시:

```text
Deployment replicas: 3
현재 Pod: 2
→ Pod 1개 추가 생성
```

## Worker Node

실제 Application Pod 실행 위치.

### kubelet

각 Node의 Agent.

API Server에서 전달받은 Pod 명세를 기준으로 컨테이너 실행 상태 관리.

### Container Runtime

실제 컨테이너 실행.

대표적으로 `containerd` 사용.

### kube-proxy

Service 트래픽을 Pod로 전달하기 위한 네트워크 규칙 관리.

## 요청 흐름

```text
kubectl apply
    ↓
kube-apiserver
    ↓
etcd에 상태 저장
    ↓
controller가 필요한 Pod 생성
    ↓
scheduler가 Node 선택
    ↓
kubelet이 컨테이너 실행
```

## 다음

`03-Cluster-and-Node`에서 Cluster와 Node 관계 정리.
