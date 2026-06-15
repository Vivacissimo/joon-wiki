---
title: SetUp
description: 
published: true
date: 2026-06-15T03:39:26.347Z
tags: 
editor: markdown
dateCreated: 2026-06-15T03:34:46.002Z
---

# Kubernetes kubeadm 클러스터 구축 정리

<br>

## Master Node 역할

Master Node는 Kubernetes Control Plane 역할을 담당한다.

```text
master
├── kube-apiserver
├── etcd
├── kube-scheduler
├── kube-controller-manager
├── kubectl
└── cluster 관리 명령 실행 위치
```

Master Node에서는 주로 아래 명령을 실행한다.

```bash
kubectl get nodes
kubectl get pods -A
kubectl apply -f <manifest.yaml>
helm install ...
kubeadm token create --print-join-command
```

<br>

## Worker Node 역할

Worker Node는 실제 Pod가 실행되는 서버이다.

```text
node1 ~ node5
├── kubelet
├── containerd
├── kube-proxy
└── Application Pod 실행
```

Worker Node는 Master에 Join된 이후 Control Plane의 스케줄링에 따라 Pod를 실행한다.

<br>

## Node 공통 사전 설정

아래 작업은 Master와 Worker 전체에서 공통으로 진행한다.

```text
master
node1
node2
node3
node4
node5
```

<br>

## 서버 기본 정보 확인

각 노드에서 현재 서버 정보를 확인한다.

```bash
hostname
whoami
cat /etc/os-release
nproc
free -h
df -h
hostname -I
ip -4 addr
```

확인해야 할 내용은 아래와 같다.

```text
OS 종류
CPU 개수
Memory 용량
Disk 용량
Private IP
현재 Hostname
```

<br>

## Hostname 설정

각 노드의 Hostname을 짧고 명확하게 설정한다.

Master Node:

```bash
sudo hostnamectl set-hostname master
```

Worker Node:

```bash
sudo hostnamectl set-hostname node1
sudo hostnamectl set-hostname node2
sudo hostnamectl set-hostname node3
sudo hostnamectl set-hostname node4
sudo hostnamectl set-hostname node5
```

변경 확인:

```bash
hostname
```

프롬프트에 바로 반영되지 않으면 SSH를 재접속한다.

```bash
exit
ssh <user>@<server-ip>
```

<br>

## Hostname을 바꾸는 이유

Hostname 변경은 Kubernetes 필수 조건은 아니다.

하지만 kubeadm은 기본적으로 현재 Hostname을 Kubernetes Node 이름으로 사용한다.

Hostname이 길면 아래와 같이 노드 관리가 불편해진다.

```text
kt-cloud-lab-nomoresh-1781244611599-node-3.novalocal
```

짧게 바꾸면 노드 상태 확인과 장애 분석이 쉬워진다.

```text
master
node1
node2
node3
node4
node5
```

<br>

## /etc/hosts 설정

모든 노드에서 동일하게 `/etc/hosts`를 설정한다.

```bash
sudo vi /etc/hosts
```

아래 내용을 추가한다.

```text
10.0.1.25 master
10.0.3.165 node1
10.0.1.183 node2
10.0.2.8 node3
10.0.2.61 node4
10.0.1.157 node5
```

<br>

## /etc/hosts를 설정하는 이유

노드 간 통신 시 IP 대신 이름을 사용할 수 있게 하기 위함이다.

예시:

```bash
ping node1
ping node2
ssh node3
```

Kubernetes 구성 자체는 IP만으로도 가능하지만, 직접 서버 6대를 관리하는 환경에서는 `/etc/hosts`를 설정하는 것이 편하다.

<br>

## 노드 간 통신 확인

Master Node에서 각 Worker Node로 통신을 확인한다.

```bash
ping -c 2 node1
ping -c 2 node2
ping -c 2 node3
ping -c 2 node4
ping -c 2 node5
```

정상 예시:

```text
2 packets transmitted, 2 received, 0% packet loss
```

<br>

## Swap 비활성화

모든 노드에서 Swap을 비활성화한다.

```bash
sudo swapoff -a
```

영구 비활성화:

```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

확인:

```bash
free -h
```

정상 상태:

```text
Swap: 0B
```

<br>

## Swap을 끄는 이유

Kubernetes는 각 노드의 Memory 상태를 기준으로 Pod를 스케줄링하고 관리한다.

Swap이 켜져 있으면 kubelet이 실제 Memory 압박 상태를 정확히 판단하기 어렵다.

따라서 kubeadm 기반 Kubernetes 구성에서는 Swap을 비활성화하는 것이 기본이다.

<br>

## Kubernetes 공통 설치 작업

아래 작업은 Master와 Worker 전체에서 공통으로 진행한다.

```text
master
node1
node2
node3
node4
node5
```

<br>

## Kernel Module 설정

Kubernetes 네트워크 구성을 위해 필요한 Kernel Module을 등록한다.

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

즉시 로드한다.

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

확인:

```bash
lsmod | grep overlay
lsmod | grep br_netfilter
```

<br>

## overlay

`overlay`는 containerd가 컨테이너 파일시스템을 효율적으로 관리하기 위해 사용하는 OverlayFS와 관련된 Kernel Module이다.

<br>

## br_netfilter

`br_netfilter`는 Linux Bridge를 지나는 패킷이 iptables 규칙을 타도록 하기 위해 필요하다.

Kubernetes의 Pod Network와 Service Network가 정상 동작하려면 필요하다.

<br>

## sysctl 설정

Kubernetes 네트워크 트래픽이 정상적으로 포워딩되도록 설정한다.

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

적용:

```bash
sudo sysctl --system
```

확인:

```bash
cat /proc/sys/net/ipv4/ip_forward
sysctl net.ipv4.ip_forward
```

정상 값:

```text
1
```

또는:

```text
net.ipv4.ip_forward = 1
```

<br>

## ip_forward를 켜는 이유

Pod와 Pod, Pod와 Service, 노드 간 트래픽을 전달하기 위해 필요하다.

`ip_forward`가 `0`이면 kubeadm init 또는 join 시 아래와 같은 에러가 발생할 수 있다.

```text
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]:
/proc/sys/net/ipv4/ip_forward contents are not set to 1
```

해결:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

<br>

## containerd 설치

모든 노드에 Container Runtime인 containerd를 설치한다.

Rocky Linux 기준:

```bash
sudo dnf install -y containerd
```

설정 파일 생성:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
```

Systemd Cgroup 사용 설정:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

재시작 및 자동 실행 설정:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
```

정상 상태:

```text
active (running)
```

<br>

## SystemdCgroup을 true로 설정하는 이유

kubelet과 containerd의 cgroup driver를 systemd 기준으로 맞추기 위해서이다.

cgroup driver가 맞지 않으면 kubelet과 container runtime 사이에서 리소스 관리 문제가 발생할 수 있다.

<br>

## kubeadm / kubelet / kubectl 설치

Kubernetes 패키지 저장소를 추가한다.

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/repodata/repomd.xml.key
EOF
```

패키지 설치:

```bash
sudo dnf install -y kubelet kubeadm kubectl
```

kubelet 활성화:

```bash
sudo systemctl enable --now kubelet
```

설치 확인:

```bash
kubeadm version
kubectl version --client
kubelet --version
```

<br>

## 각 패키지 역할

| 패키지 | 역할 |
|---|---|
| kubeadm | 클러스터 초기화 및 Worker Node Join |
| kubelet | 노드에서 Pod 실행 상태 관리 |
| kubectl | Kubernetes API Server에 명령 전달 |

<br>

## kubelet 상태 확인

```bash
sudo systemctl status kubelet
```

초기에는 kubelet이 완전히 정상 상태가 아닐 수 있다.

kubeadm init 또는 join 전에는 kubelet이 아직 클러스터 설정을 받지 않았기 때문이다.

따라서 설치 직후 kubelet 관련 경고가 일부 보일 수 있다.

<br>

## Master Node 초기화

이 작업은 Master Node에서만 진행한다.

```text
master
```

<br>

## Master IP 설정

현재 Master Node의 Private IP는 아래와 같다.

```text
10.0.1.25
```

환경변수로 지정한다.

```bash
MASTER_IP=10.0.1.25
echo $MASTER_IP
```

<br>

## kubeadm 이미지 Pull

kubeadm init 전에 필요한 이미지를 미리 Pull 한다.

```bash
sudo kubeadm config images pull
```

에러가 발생하면 DNS, 인터넷 연결, 방화벽 설정을 확인한다.

<br>

## kubeadm init 실행

Master Node를 Kubernetes Control Plane으로 초기화한다.

```bash
sudo kubeadm init \
  --node-name master \
  --apiserver-advertise-address=$MASTER_IP \
  --pod-network-cidr=192.168.0.0/16
```

<br>

## kubeadm init 옵션 설명

| 옵션 | 설명 |
|---|---|
| `--node-name master` | Kubernetes Node 이름을 `master`로 지정 |
| `--apiserver-advertise-address=$MASTER_IP` | API Server가 사용할 Master Private IP 지정 |
| `--pod-network-cidr=192.168.0.0/16` | Pod들이 사용할 내부 IP 대역 지정 |

<br>

## Pod Network CIDR

`192.168.0.0/16`은 서버 IP 대역이 아니다.

Kubernetes 내부 Pod들이 사용할 IP 대역이다.

이번 서버들의 Node IP는 `10.0.x.x` 대역이다.

```text
Node Network = 10.0.x.x
Pod Network  = 192.168.0.0/16
```

Node IP 대역과 Pod IP 대역은 겹치지 않는 것이 좋다.

Calico CNI를 사용할 때 `192.168.0.0/16`은 자주 사용하는 Pod CIDR이다.

<br>

## kubeadm init 성공 시 출력

성공하면 마지막에 Worker Node Join 명령이 출력된다.

예시:

```bash
kubeadm join 10.0.1.25:6443 --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

이 명령은 Worker Node를 클러스터에 붙일 때 필요하므로 복사해둔다.

놓쳤을 경우 Master Node에서 다시 생성할 수 있다.

```bash
kubeadm token create --print-join-command
```

<br>

## kubectl 설정

kubeadm init 이후 일반 사용자 계정에서 kubectl을 사용할 수 있도록 설정한다.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

확인:

```bash
kubectl get nodes
```

초기 상태:

```text
NAME     STATUS     ROLES           VERSION
master   NotReady   control-plane   v1.36.2
```

처음에는 `NotReady`가 정상이다.

아직 CNI가 설치되지 않았기 때문이다.

<br>

## Control Plane Pod 확인

```bash
kubectl get pods -n kube-system
```

정상 예시:

```text
etcd-master                       Running
kube-apiserver-master             Running
kube-controller-manager-master    Running
kube-scheduler-master             Running
kube-proxy-xxxxx                  Running
coredns-xxxxx                     Pending
```

CoreDNS는 CNI 설치 전까지 Pending일 수 있다.

<br>

## Calico CNI 설치

이 작업은 Master Node에서만 실행한다.

```text
master
```

<br>

## Calico를 설치하는 이유

Kubernetes는 기본적으로 Pod Network를 직접 구성하지 않는다.

Pod끼리 서로 통신하려면 CNI 플러그인이 필요하다.

이번 클러스터에서는 Calico를 CNI로 사용한다.

Calico는 아래 역할을 담당한다.

```text
Pod Network 구성
Node 간 Pod 통신
NetworkPolicy 적용
```

심화 프로젝트에서는 NetworkPolicy도 사용하므로 Calico를 사용한다.

<br>

## Tigera Operator 설치

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/tigera-operator.yaml
```

생성되는 주요 리소스:

```text
namespace/tigera-operator
serviceaccount/tigera-operator
clusterrole
clusterrolebinding
deployment/tigera-operator
```

확인:

```bash
kubectl get pods -n tigera-operator
```

정상:

```text
tigera-operator-xxxxx   1/1   Running
```

<br>

## CRD 확인

Tigera Operator 설치 직후에는 CRD가 아직 완전히 등록되지 않았을 수 있다.

확인:

```bash
kubectl get crd | grep tigera
```

정상적으로 아래와 같은 CRD가 보여야 한다.

```text
installations.operator.tigera.io
apiservers.operator.tigera.io
goldmanes.operator.tigera.io
whiskers.operator.tigera.io
```

CRD 등록을 기다린다.

```bash
kubectl wait --for=condition=Established --timeout=120s crd/installations.operator.tigera.io
kubectl wait --for=condition=Established --timeout=120s crd/apiservers.operator.tigera.io
```

<br>

## Custom Resources 적용

CRD가 등록된 뒤 custom-resources를 적용한다.

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
```

이미 일부 리소스가 있으면 `apply`를 사용한다.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
```

<br>

## Calico CRD 관련 에러

아래와 같은 에러가 발생할 수 있다.

```text
no matches for kind "Installation" in version "operator.tigera.io/v1"
ensure CRDs are installed first
```

원인은 Tigera Operator가 CRD를 등록하기 전에 custom-resources를 너무 빨리 적용했기 때문이다.

해결 방법:

```bash
kubectl get crd | grep tigera
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
```

<br>

## Calico 상태 확인

```bash
kubectl get pods -A
kubectl get pods -n calico-system -o wide
```

정상 예시:

```text
calico-node-xxxxx              Running
calico-kube-controllers-xxxxx  Running
calico-typha-xxxxx             Running
```

<br>

## Master Ready 확인

```bash
kubectl get nodes
```

정상:

```text
NAME     STATUS   ROLES           VERSION
master   Ready    control-plane   v1.36.2
```

CNI가 정상 설치되면 Master Node가 `NotReady`에서 `Ready`로 변경된다.

<br>

## Worker Node Join

이 작업은 Worker Node에서 진행한다.

```text
node1
node2
node3
node4
node5
```

<br>

## Master에서 Join 명령 생성

Master Node에서 실행한다.

```bash
kubeadm token create --print-join-command
```

출력 예시:

```bash
kubeadm join 10.0.1.25:6443 --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

이 명령을 각 Worker Node에서 실행한다.

<br>

## Worker Node 사전 확인

각 Worker Node에서 아래 설정이 완료되어 있어야 한다.

```text
hostname 설정
/etc/hosts 설정
swap off
kernel module 설정
sysctl 설정
containerd 설치
kubeadm/kubelet/kubectl 설치
```

특히 `ip_forward`를 확인한다.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

정상:

```text
1
```

`0`이면 아래 명령으로 수정한다.

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

<br>

## node1 Join

node1에서 실행한다.

```bash
sudo kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --node-name node1
```

<br>

## node2 Join

node2에서 실행한다.

```bash
sudo kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --node-name node2
```

<br>

## node3 Join

node3에서 실행한다.

```bash
sudo kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --node-name node3
```

<br>

## node4 Join

node4에서 실행한다.

```bash
sudo kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --node-name node4
```

<br>

## node5 Join

node5에서 실행한다.

```bash
sudo kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --node-name node5
```

<br>

## Join 성공 메시지

정상적으로 Join되면 아래와 같은 메시지가 출력된다.

```text
This node has joined the cluster:

* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

이 메시지가 나오면 Worker Node Join은 성공한 것이다.

<br>

## Join 실패 시 초기화

이미 Join을 시도하다 실패한 노드에서 다시 Join할 때 아래와 같은 에러가 날 수 있다.

```text
/etc/kubernetes/kubelet.conf already exists
port 10250 is in use
```

해당 Worker Node에서 초기화한다.

```bash
sudo kubeadm reset -f
sudo systemctl restart kubelet
```

Master에 잘못 등록된 Node가 남아 있으면 Master에서 삭제한다.

```bash
kubectl delete node node1
```

그 후 Worker Node에서 Join 명령을 다시 실행한다.

<br>

## Kubernetes 클러스터 상태 확인

Master Node에서 실행한다.

```bash
kubectl get nodes -o wide
```

최종 정상 상태:

```text
NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP
master   Ready    control-plane   ...   v1.36.2   10.0.1.25
node1    Ready    <none>          ...   v1.36.2   10.0.3.165
node2    Ready    <none>          ...   v1.36.2   10.0.1.183
node3    Ready    <none>          ...   v1.36.2   10.0.2.8
node4    Ready    <none>          ...   v1.36.2   10.0.2.61
node5    Ready    <none>          ...   v1.36.2   10.0.1.157
```

<br>

## 전체 Pod 상태 확인

```bash
kubectl get pods -A -o wide
```

확인 대상:

```text
kube-system
calico-system
tigera-operator
```

<br>

## kube-system 확인

```bash
kubectl get pods -n kube-system -o wide
```

정상 예시:

```text
coredns-xxxxx                  Running
etcd-master                    Running
kube-apiserver-master          Running
kube-controller-manager-master Running
kube-proxy-xxxxx               Running
kube-scheduler-master          Running
```

<br>

## calico-system 확인

```bash
kubectl get pods -n calico-system -o wide
```

정상 예시:

```text
calico-node-xxxxx              Running
calico-kube-controllers-xxxxx  Running
calico-typha-xxxxx             Running
```

각 Worker Node마다 `calico-node`가 하나씩 떠야 한다.

<br>

## Node가 NotReady일 때 확인

```bash
kubectl describe node node2
```

`Conditions` 영역을 확인한다.

자주 나오는 원인:

```text
NetworkPluginNotReady
container runtime network not ready
cni plugin not initialized
```

이 경우 Calico CNI가 아직 적용 중이거나 문제가 있는 상태이다.

Calico Pod 상태를 확인한다.

```bash
kubectl get pods -n calico-system -o wide
```

<br>

## Master Taint 확인

Master Node에는 일반 Pod가 배치되지 않도록 Taint가 걸려 있어야 한다.

```bash
kubectl describe node master | grep -i taint
```

정상 예시:

```text
node-role.kubernetes.io/control-plane:NoSchedule
```

없으면 추가한다.

```bash
kubectl taint nodes master node-role.kubernetes.io/control-plane=:NoSchedule
```

<br>

## Cluster 기본 구성 완료 기준

아래 조건을 만족하면 Kubernetes 기본 클러스터 구성은 완료이다.

```text
master Ready
node1 Ready
node2 Ready
node3 Ready
node4 Ready
node5 Ready
kube-system Pod Running
calico-system Pod Running
CoreDNS Running
```

<br>

## 자주 발생한 문제와 해결

## kubeadm init 또는 join 시 ip_forward 에러

에러:

```text
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]:
/proc/sys/net/ipv4/ip_forward contents are not set to 1
```

원인:

```text
net.ipv4.ip_forward 값이 1이 아님
```

확인:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

해결:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

영구 설정:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

<br>

## Calico custom-resources 적용 시 CRD 에러

에러:

```text
no matches for kind "Installation" in version "operator.tigera.io/v1"
ensure CRDs are installed first
```

원인:

```text
Tigera Operator가 CRD를 등록하기 전에 custom-resources.yaml을 적용함
```

해결:

```bash
kubectl get crd | grep tigera
```

CRD가 보이면 다시 적용한다.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
```

<br>

## Worker Join 성공 후 NotReady

현상:

```text
node2 NotReady
node3 NotReady
```

원인:

```text
Join 직후 Calico CNI가 아직 적용 중일 수 있음
```

확인:

```bash
kubectl get pods -n calico-system -o wide
kubectl describe node node2
```

대부분 몇 분 기다리면 Ready로 전환된다.

<br>

## kernel unsupported warning

경고:

```text
[WARNING SystemVerification]:
kernel release 5.14.0-611.11.1.el9_7.x86_64 is unsupported.
```

원인:

```text
kubeadm에서 권장하는 LTS Kernel 목록과 Rocky Linux 9.7 Kernel 버전이 다르기 때문에 발생
```

이 경고는 Fatal Error가 아니므로 init 또는 join이 성공하면 진행 가능하다.

<br>

## join 재시도 시 기존 파일 에러

에러 예시:

```text
/etc/kubernetes/kubelet.conf already exists
port 10250 is in use
```

해결:

Worker Node에서 초기화한다.

```bash
sudo kubeadm reset -f
sudo systemctl restart kubelet
```

Master에 기존 Node가 남아 있으면 삭제한다.

```bash
kubectl delete node <node-name>
```

그 후 다시 Join한다.

<br>

## 현재 완료된 상태

```text
외부 서버 6대 준비 완료
Master Node 구성 완료
Worker Node 5대 Join 완료
Calico CNI 구성 완료
전체 Node Ready
```

<br>

## 다음 단계

Kubernetes 기본 클러스터 구성이 끝났으므로 심화 프로젝트 도구를 설치한다.

```text
1. prod Namespace 생성
2. Metrics Server 설치
3. Helm 설치 확인
4. Istio 설치
5. prod Namespace에 Istio Sidecar Injection 활성화
6. ArgoCD 설치
7. Kiali 설치
8. Prometheus / Grafana 설치
9. Jaeger 설치
10. NoMoreSh 서비스 배포
```

<br>

## prod Namespace 생성

```bash
kubectl create namespace prod
```

이미 존재하면 문제 없다.

확인:

```bash
kubectl get ns
```

<br>

## Istio Sidecar Injection 라벨 설정

```bash
kubectl label namespace prod istio-injection=enabled --overwrite
```

확인:

```bash
kubectl get namespace prod --show-labels
```

정상 예시:

```text
prod   istio-injection=enabled
```

<br>

## 주의 사항

이번 클러스터는 AWS EKS가 아니라 외부 서버 기반 kubeadm 클러스터이다.

따라서 AWS ECR Private Image를 사용할 경우 EKS처럼 IAM Role로 자동 Pull되지 않는다.

외부 서버에서 ECR 이미지를 Pull하려면 별도 인증이 필요하다.

```text
EKS
→ Node IAM Role로 ECR Pull 가능

외부 kubeadm 클러스터
→ ECR Pull Secret 필요 가능
```

이 부분은 NoMoreSh 배포 단계에서 별도로 확인해야 한다.