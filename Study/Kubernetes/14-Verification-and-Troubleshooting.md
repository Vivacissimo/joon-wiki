---
title: 14. Verification and Troubleshooting
description: Kubernetes Cluster 최종 확인과 기본 문제 해결
published: true
tags: kubernetes,troubleshooting
editor: markdown
---

# Cluster 검증과 Troubleshooting

설치 완료 후 Node, System Pod, CNI, DNS 순서로 확인.

## Node 확인

```bash
kubectl get nodes -o wide
```

정상 기준:

```text
Control Plane → Ready
Worker Node   → Ready
```

## 전체 Pod 확인

```bash
kubectl get pods -A -o wide
```

주요 Namespace:

- kube-system
- calico-system
- tigera-operator

## kube-system

```bash
kubectl get pods -n kube-system -o wide
```

주요 Pod:

```text
coredns
etcd
kube-apiserver
kube-controller-manager
kube-proxy
kube-scheduler
```

## Calico

```bash
kubectl get pods -n calico-system -o wide
```

각 Node에 필요한 Calico 구성 요소가 `Running`인지 확인.

## Control Plane Taint

```bash
kubectl describe node master | grep -i taint
```

일반적인 kubeadm 구성:

```text
node-role.kubernetes.io/control-plane:NoSchedule
```

## Node가 NotReady

```bash
kubectl describe node <node-name>
```

자주 확인할 메시지:

```text
NetworkPluginNotReady
container runtime network not ready
cni plugin not initialized
```

추가 확인:

```bash
systemctl status kubelet
systemctl status containerd
kubectl get pods -n calico-system -o wide
```

## ip_forward 에러

확인:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

정상:

```text
1
```

수정:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl --system
```

## Join 재시도 에러

예시:

```text
/etc/kubernetes/kubelet.conf already exists
port 10250 is in use
```

Worker Node 초기화:

```bash
sudo kubeadm reset -f
sudo systemctl restart kubelet
```

필요하면 Control Plane에서 기존 Node 삭제:

```bash
kubectl delete node <node-name>
```

## Pod 문제 확인 순서

```bash
kubectl get pods -A
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl get events -A --sort-by=.lastTimestamp
```

## 기본 완료 기준

```text
모든 Node Ready
Control Plane Pod Running
CoreDNS Running
CNI Pod Running
Worker Node Join 완료
Pod 간 Network 정상
```

여기까지 완료되면 kubeadm 기반 Kubernetes 기본 Cluster 구성 완료.

다음 학습은 Deployment/Service 실습, Ingress, Helm, ArgoCD, HPA, NetworkPolicy 순서로 확장.
