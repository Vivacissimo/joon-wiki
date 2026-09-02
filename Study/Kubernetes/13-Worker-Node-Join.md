---
title: 13. Worker Node Join
description: Worker Node를 Cluster에 연결
published: true
tags: kubernetes,kubeadm
editor: markdown
---

# Worker Node Join

Worker Node에서 실행.

## Join 명령 생성

Control Plane에서 실행:

```bash
kubeadm token create --print-join-command
```

예시:

```bash
kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

## Worker 사전 확인

다음 설정이 먼저 끝나 있어야 함.

- Hostname
- Swap Off
- Kernel Module
- sysctl
- containerd
- kubeadm / kubelet

특히 확인:

```bash
cat /proc/sys/net/ipv4/ip_forward
systemctl status containerd
```

## Join 실행

```bash
sudo kubeadm join 10.0.1.25:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --node-name node1
```

다른 Worker도 같은 방식으로 Join.

`--node-name`만 각 Node 이름에 맞게 변경.

## 등록 확인

Control Plane에서:

```bash
kubectl get nodes -o wide
```

예시:

```text
NAME     STATUS   ROLES           INTERNAL-IP
master   Ready    control-plane   10.0.1.25
node1    Ready    <none>          10.0.3.165
node2    Ready    <none>          10.0.1.183
```

## Join 재시도

기존 Join이 실패한 Node에서 설정이 남아 있을 수 있음.

```bash
sudo kubeadm reset -f
sudo systemctl restart kubelet
```

Control Plane에 잘못 등록된 Node가 남았다면:

```bash
kubectl delete node <node-name>
```

그 후 다시 Join.

## Join 후 NotReady

Join 직후 CNI 적용이 끝나지 않아 잠시 `NotReady`일 수 있음.

확인:

```bash
kubectl describe node <node-name>
kubectl get pods -n calico-system -o wide
```

## 다음

`14-Verification-and-Troubleshooting`에서 전체 Cluster 정상 여부 확인.
