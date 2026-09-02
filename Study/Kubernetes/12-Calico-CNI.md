---
title: 12. Calico CNI
description: Calico CNI 설치와 확인
published: true
tags: kubernetes,calico,cni
editor: markdown
---

# Calico CNI

Kubernetes는 Pod Network를 직접 구현하지 않음.

Pod 간 통신을 위해 CNI 플러그인 필요.

이번 구성에서는 Calico 사용.

## Calico 역할

- Pod IP 할당
- Pod 간 통신
- Node 간 Pod 통신
- NetworkPolicy 적용

## Tigera Operator 설치

버전은 설치 시점의 Calico 공식 문서 기준으로 확인.

예시:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/tigera-operator.yaml
```

확인:

```bash
kubectl get pods -n tigera-operator
```

## CRD 확인

```bash
kubectl get crd | grep tigera
```

필요하면 CRD 등록 완료까지 대기.

```bash
kubectl wait --for=condition=Established \
  --timeout=120s \
  crd/installations.operator.tigera.io
```

## Custom Resources 적용

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
```

`kubeadm init`의 `--pod-network-cidr`와 Calico IP Pool CIDR이 맞는지 확인.

## 상태 확인

```bash
kubectl get pods -A
kubectl get pods -n calico-system -o wide
kubectl get nodes
```

정상 예시:

```text
calico-node-*               Running
calico-kube-controllers-*   Running
calico-typha-*              Running
```

CNI 구성이 끝나면 Control Plane Node가 `Ready`로 변경.

## CRD 에러

예시:

```text
no matches for kind "Installation"
ensure CRDs are installed first
```

Tigera Operator가 CRD 등록을 끝내기 전에 Custom Resource를 적용한 경우 발생 가능.

확인 후 다시 적용.

```bash
kubectl get crd | grep tigera
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
```

## 다음

`13-Worker-Node-Join`에서 Worker Node를 Cluster에 연결.
