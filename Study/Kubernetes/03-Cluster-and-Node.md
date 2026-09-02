---
title: 03. Cluster and Node
description: Cluster와 Node 개념
published: true
tags: kubernetes
editor: markdown
---

# Cluster와 Node

## Cluster

여러 Node를 하나의 Kubernetes 환경으로 묶은 단위.

```text
Cluster
├── Control Plane
├── Worker Node 1
├── Worker Node 2
└── Worker Node 3
```

## Node

Kubernetes가 사용하는 서버 한 대.

물리 서버, VM, Cloud Instance 모두 가능.

```text
AWS EC2
KT Cloud VM
Bare Metal
Local VM
→ Kubernetes Node로 사용 가능
```

## Control Plane Node

클러스터 제어 역할.

주요 구성:

- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager

일반적인 구성에서는 Application Pod를 Worker Node에 배치.

## Worker Node

실제 Application Pod 실행.

주요 구성:

- kubelet
- containerd
- kube-proxy
- CNI

## Node와 Pod 관계

```text
Worker Node 1
├── Pod A
├── Pod B
└── Pod C

Worker Node 2
├── Pod D
└── Pod E
```

Pod가 특정 Node에 고정되는 것은 기본 동작이 아님.

Scheduler가 조건에 맞는 Node를 선택.

## Node 확인

```bash
kubectl get nodes
kubectl get nodes -o wide
```

상태 예시:

```text
NAME     STATUS   ROLES           VERSION
master   Ready    control-plane   v1.x.x
node1    Ready    <none>          v1.x.x
node2    Ready    <none>          v1.x.x
```

## Ready / NotReady

`Ready`
: 정상적으로 Pod 실행 가능

`NotReady`
: kubelet, CNI, Runtime, Node Network 등 문제 가능

## 다음

`04-Pod`에서 Kubernetes의 가장 작은 실행 단위 확인.
