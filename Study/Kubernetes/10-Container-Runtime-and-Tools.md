---
title: 10. Container Runtime and Tools
description: containerd kubeadm kubelet kubectl 설치
published: true
tags: kubernetes,kubeadm,containerd
editor: markdown
---

# Container Runtime과 Kubernetes 도구 설치

Control Plane과 Worker Node 모두 동일하게 진행.

## containerd 설치

```bash
sudo dnf install -y containerd
```

기본 설정 생성:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
```

Systemd Cgroup 사용:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

재시작:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
```

정상 상태:

```text
active (running)
```

## SystemdCgroup

kubelet과 containerd의 cgroup driver를 `systemd` 기준으로 맞추기 위한 설정.

## Kubernetes Repository

설치하려는 Kubernetes Minor Version에 맞춰 Repository 지정.

예시:

```bash
K8S_MINOR=v1.36

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/${K8S_MINOR}/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/${K8S_MINOR}/rpm/repodata/repomd.xml.key
EOF
```

## 패키지 설치

```bash
sudo dnf install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

## 역할

| 도구 | 역할 |
|---|---|
| kubeadm | Cluster 초기화 / Node Join |
| kubelet | 각 Node의 Pod 실행 상태 관리 |
| kubectl | API Server에 명령 전달 |

## 설치 확인

```bash
kubeadm version
kubelet --version
kubectl version --client
```

`kubeadm init` 또는 `join` 전에는 kubelet 경고가 보일 수 있음.

아직 Cluster 설정을 받지 않은 상태이기 때문.

## 다음

`11-Control-Plane-Setup`에서 첫 Control Plane Node 초기화.
