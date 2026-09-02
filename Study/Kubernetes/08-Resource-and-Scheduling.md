---
title: 08. Resource and Scheduling
description: Resource와 Scheduling 기본
published: true
tags: kubernetes
editor: markdown
---

# Resource와 Scheduling

Scheduler는 Pod를 아무 Node에나 배치하지 않음.

Node 자원과 Pod 조건을 확인해서 실행 위치 결정.

## Requests

Pod가 최소한 필요하다고 요청하는 자원.

Scheduler가 Node 선택할 때 사용.

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

## Limits

Container가 사용할 수 있는 최대 자원.

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

## CPU 단위

```text
1000m = 1 CPU
500m  = 0.5 CPU
250m  = 0.25 CPU
```

## Memory 단위

```text
Mi = MiB
Gi = GiB
```

## Taint / Toleration

특정 Node에 일반 Pod가 배치되지 않도록 제한.

예시:

```text
GPU Node
→ GPU 작업만 배치
```

Taint는 Node에 적용.
Toleration은 Pod에 적용.

## NodeSelector

특정 Label이 있는 Node에 Pod 배치.

```yaml
nodeSelector:
  workload: gpu
```

## Affinity

Pod 배치 조건을 더 세밀하게 지정.

- Node Affinity
- Pod Affinity
- Pod Anti-Affinity

## Control Plane Taint

kubeadm 구성에서는 Control Plane에 일반 Pod가 배치되지 않도록 Taint가 기본 적용됨.

```bash
kubectl describe node master | grep -i taint
```

## 확인 명령

```bash
kubectl top nodes
kubectl top pods
kubectl describe node <node-name>
```

`kubectl top`은 Metrics Server 필요.

## 다음

이제 기본 개념을 바탕으로 `09-kubeadm-Prerequisites`부터 직접 클러스터 설치 진행.
