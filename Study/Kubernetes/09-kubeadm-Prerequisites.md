---
title: 09. kubeadm Prerequisites
description: kubeadm 설치 전 사전 설정
published: true
tags: kubernetes,kubeadm
editor: markdown
---

# kubeadm 사전 설정

Rocky Linux 기반 직접 구축 환경 기준.

Control Plane과 Worker Node 모두 동일하게 적용.

## 기본 확인

```bash
hostname
cat /etc/os-release
nproc
free -h
df -h
hostname -I
```

확인 항목:

- OS
- CPU
- Memory
- Disk
- Private IP
- Hostname

## Hostname

예시:

```bash
sudo hostnamectl set-hostname master
sudo hostnamectl set-hostname node1
```

kubeadm은 기본적으로 Hostname을 Node 이름으로 사용.

## /etc/hosts

직접 관리하는 서버 환경에서는 이름 기반 통신을 위해 등록해두면 편함.

```text
10.0.1.25 master
10.0.3.165 node1
10.0.1.183 node2
```

환경에 맞는 실제 Private IP 사용.

## Swap 비활성화

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

확인:

```bash
free -h
```

## Kernel Module

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

## sysctl

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

확인:

```bash
sysctl net.ipv4.ip_forward
```

정상 값:

```text
net.ipv4.ip_forward = 1
```

## 왜 필요한가

`overlay`
: containerd OverlayFS 사용

`br_netfilter`
: Bridge 트래픽을 iptables 규칙에서 처리

`ip_forward`
: Pod와 Node 사이 패킷 전달

## 노드 간 통신 확인

Control Plane에서 Worker Node로 확인.

```bash
ping -c 2 node1
ping -c 2 node2
```

## 다음

`10-Container-Runtime-and-Tools`에서 containerd와 kubeadm/kubelet/kubectl 설치.
