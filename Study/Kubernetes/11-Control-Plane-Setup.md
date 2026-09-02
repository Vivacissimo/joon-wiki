---
title: 11. Control Plane Setup
description: kubeadm Control Plane 초기화
published: true
tags: kubernetes,kubeadm
editor: markdown
---

# Control Plane 초기화

이 작업은 첫 Control Plane Node에서만 진행.

## IP 확인

```bash
hostname -I
```

예시:

```bash
MASTER_IP=10.0.1.25
```

## 이미지 Pull

```bash
sudo kubeadm config images pull
```

## kubeadm init

Calico와 `192.168.0.0/16` Pod CIDR을 사용할 경우 예시:

```bash
sudo kubeadm init \
  --node-name master \
  --apiserver-advertise-address=$MASTER_IP \
  --pod-network-cidr=192.168.0.0/16
```

## 주요 옵션

| 옵션 | 의미 |
|---|---|
| `--node-name` | Kubernetes Node 이름 |
| `--apiserver-advertise-address` | API Server가 사용할 Node IP |
| `--pod-network-cidr` | Pod에 할당할 내부 IP 대역 |

## Network CIDR

```text
Node Network → 실제 서버 네트워크
Pod Network  → Pod 전용 네트워크
```

서로 겹치지 않게 구성.

## Join 명령

초기화 성공 시 Worker Node용 `kubeadm join` 명령 출력.

잃어버렸다면 다시 생성 가능.

```bash
kubeadm token create --print-join-command
```

## kubectl 설정

일반 사용자 계정에서 kubectl 사용:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

확인:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

CNI 설치 전에는 Control Plane이 `NotReady`, CoreDNS가 `Pending`일 수 있음.

## Control Plane 주요 Pod

```text
kube-apiserver
etcd
kube-scheduler
kube-controller-manager
```

확인:

```bash
kubectl get pods -n kube-system
```

## 다음

`12-Calico-CNI`에서 Pod Network 구성.
