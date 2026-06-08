---
title: EC2
description: 
published: true
date: 2026-06-08T05:42:05.455Z
tags: 
editor: markdown
dateCreated: 2026-06-08T05:26:36.586Z
---

# EKS 
<br>

## 1. EKS란?

**EKS**는 **Amazon Elastic Kubernetes Service**의 약자이다.

AWS에서 제공하는 **관리형 Kubernetes 서비스**.

직접 서버에 Kubernetes를 설치하고 Control Plane까지 운영하는 대신,  
AWS가 Kubernetes의 핵심 관리 영역을 대신 운영해주는 서비스이다.

참고: [Amazon EKS 공식 문서](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
<br>

## 2. EKS를 쉽게 이해하면

```text
EKS = AWS에서 관리해주는 Kubernetes 클러스터
EC2 = Pod가 실제로 실행되는 워커 노드
Pod = 서비스 컨테이너가 실행되는 단위
kubectl = EKS 클러스터를 제어하는 명령어
```

조금 더 풀면 아래와 같습니다.

```text
내 PC
  |
  | kubectl 명령
  v
EKS Control Plane
  |
  | Pod 배치 지시
  v
EC2 Worker Node
  |
  | 컨테이너 실행
  v
patient-service / appointment-service / prescription-service / notification-service
```
<br>

## 3. EKS의 구성

EKS는 크게 두 부분으로 나뉜다.

```text
EKS Cluster
├── Control Plane
└── Worker Node
```
<br>

## 3-1. Control Plane

**Control Plane**은 Kubernetes의 관리 영역입니다.

여기에는 아래 같은 Kubernetes 핵심 구성요소가 포함됩니다.

| 구성요소 | 역할 |
|---|---|
| API Server | kubectl 요청을 받는 Kubernetes API |
| Scheduler | Pod를 어느 Node에 배치할지 결정 |
| Controller Manager | Deployment, ReplicaSet 등 상태 관리 |
| etcd | Kubernetes 클러스터 상태 저장소 |

EKS에서는 이 Control Plane을 사용자가 직접 관리하지 않고,  
**AWS가 관리**합니다.

즉, 사용자는 Control Plane 서버에 SSH로 접속하지 않습니다.

```text
EKS Control Plane = AWS가 관리
SSH 접속 대상 아님
kubectl로 접근
```

참고: [Amazon EKS Control Plane 공식 문서](https://docs.aws.amazon.com/eks/latest/best-practices/control-plane.html)

---

## 3-2. Worker Node

**Worker Node**는 실제 애플리케이션 Pod가 실행되는 서버입니다.

EKS에서 Worker Node는 보통 **EC2 인스턴스**입니다.

```text
Worker Node = EC2
Pod 실행 위치 = EC2 위
```

예를 들어 심화 프로젝트에서 아래 서비스들이 있다면:

```text
patient-service
appointment-service
prescription-service
notification-service
```

이 서비스들의 Pod는 EKS Control Plane에 직접 뜨는 것이 아니라,  
**EKS 클러스터에 연결된 EC2 워커 노드 위에서 실행**됩니다.

참고: [Amazon EKS Managed Node Groups 공식 문서](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
<br>

## 4. EKS와 EC2의 차이

| 구분 | 설명 |
|---|---|
| EKS | Kubernetes 클러스터 관리 서비스 |
| EC2 | 실제 서버 역할을 하는 가상 머신 |
| Control Plane | AWS가 관리하는 Kubernetes 관리 영역 |
| Worker Node | Pod가 실행되는 EC2 서버 |
| SSH 접속 | EC2 워커 노드에 가능 |
| kubectl 접근 | EKS 클러스터 API 서버에 접근 |

핵심은 아래와 같습니다.

```text
EKS = 클러스터 관리 영역
EC2 = 실제 작업 서버
```
<br>

## 5. 접근 방식
<br>

## 5-1. EKS 접근

EKS는 SSH로 접속하지 않습니다.

EKS는 `kubectl`로 접근합니다.

```bash
aws eks update-kubeconfig --region ap-northeast-2 --name <클러스터명>

kubectl get nodes
kubectl get pods -A
```

`aws eks update-kubeconfig` 명령은 내 PC의 kubeconfig 파일에 EKS 클러스터 접속 정보를 추가합니다.

그 이후 `kubectl`은 이 kubeconfig 정보를 이용해서 EKS API 서버와 통신합니다.

참고: [Amazon EKS kubeconfig 생성 공식 문서](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)
<br>

## 5-2. EC2 워커 노드 접근

EC2 워커 노드는 필요하면 SSH로 접근할 수 있습니다.

```bash
ssh -i key.pem ec2-user@<EC2_PUBLIC_IP>
```

다만 EKS에서는 보통 EC2에 직접 SSH로 들어가기보다,  
아래 명령어로 Kubernetes 관점에서 확인하는 경우가 많습니다.

```bash
kubectl get nodes
kubectl describe node <노드명>
kubectl get pods -A -o wide
kubectl logs -n <namespace> <pod명>
kubectl exec -it -n <namespace> <pod명> -- /bin/sh
```
<br>

## 6. EKS에서 애플리케이션이 배포되는 흐름

심화 프로젝트 기준으로 보면 흐름은 아래와 같습니다.

```text
1. 개발자가 GitHub에 코드 push
2. GitHub Actions가 Docker 이미지 빌드
3. Container Registry에 이미지 push
4. ArgoCD가 Git 변경사항 감지
5. EKS 클러스터에 배포 반영
6. EC2 워커 노드 위에 Pod 생성
7. Service / Ingress / LoadBalancer를 통해 외부 접근
```

구조로 보면 아래와 같습니다.

```text
GitHub
  |
  v
GitHub Actions
  |
  v
Container Registry
  |
  v
ArgoCD
  |
  v
EKS Cluster
  |
  v
EC2 Worker Node
  |
  v
Medical Service Pods
```
<br>

## 7. EKS에서 자주 나오는 용어

| 용어 | 의미 |
|---|---|
| Cluster | Kubernetes 전체 묶음 |
| Control Plane | 클러스터를 관리하는 영역 |
| Node | Pod가 실행되는 서버 |
| Managed Node Group | AWS가 관리해주는 EC2 워커 노드 그룹 |
| Pod | 컨테이너 실행 단위 |
| Deployment | Pod를 원하는 개수만큼 유지하는 리소스 |
| Service | Pod 접근을 위한 내부 네트워크 주소 |
| Ingress | 외부 HTTP/HTTPS 요청을 서비스로 연결 |
| LoadBalancer | 외부 접속을 받는 AWS 로드밸런서 |
| kubeconfig | kubectl이 클러스터에 접속할 때 쓰는 설정 파일 |
| IAM | AWS 권한 관리 |
| VPC | EKS와 EC2가 들어가는 AWS 네트워크 |
<br>

## 8. EC2를 매일 꺼도 되는 이유

EKS에서 EC2 워커 노드는 **Managed Node Group**으로 관리할 수 있습니다.

이 경우 EC2를 직접 하나씩 끄는 것보다,  
노드 그룹의 개수를 줄이는 방식이 좋습니다.

실습 끝날 때는 노드 개수를 0개로 줄입니다.

```bash
eksctl scale nodegroup \
  --cluster <클러스터명> \
  --name <노드그룹명> \
  --nodes 0
```

다시 실습할 때는 노드 개수를 다시 올립니다.

```bash
eksctl scale nodegroup \
  --cluster <클러스터명> \
  --name <노드그룹명> \
  --nodes 3
```

이렇게 하면 EC2 워커 노드 비용을 줄일 수 있습니다.

참고: [Amazon EKS Managed Node Groups 공식 문서](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
<br>

## 9. 비용 관점에서 중요한 점

EC2 워커 노드를 0개로 줄이면 EC2 비용은 줄일 수 있습니다.

하지만 아래 리소스는 남아 있으면 계속 비용이 발생할 수 있습니다.

| 리소스 | 비용 발생 여부 |
|---|---|
| EKS Control Plane | 클러스터가 살아 있으면 발생 |
| EC2 Worker Node | 노드가 켜져 있으면 발생 |
| EBS Volume | 볼륨이 남아 있으면 발생 |
| Load Balancer | 삭제하지 않으면 발생 |
| NAT Gateway | 사용 중이면 발생 |

즉, EC2만 끄면 EC2 비용은 줄어들지만,  
EKS 클러스터 자체 비용은 계속 남습니다.

```text
EC2만 끄면 EC2 비용은 줄어듦
하지만 EKS 클러스터 자체 비용은 남음
```

참고: [Amazon EKS 가격 공식 페이지](https://aws.amazon.com/eks/pricing/)
<br>

## 10. 프로젝트 기준으로 이해하기

심화 프로젝트에서 EKS를 쓴다면 구조는 아래와 같이 보면 됩니다.

```text
AWS
└── EKS Cluster
    ├── Control Plane
    │   └── AWS가 관리
    │
    ├── Managed Node Group
    │   ├── EC2 Worker Node 1
    │   ├── EC2 Worker Node 2
    │   └── EC2 Worker Node 3
    │
    ├── medical namespace
    │   ├── patient-service
    │   ├── appointment-service
    │   ├── prescription-service
    │   └── notification-service
    │
    ├── argocd namespace
    │   └── ArgoCD
    │
    ├── istio-system namespace
    │   └── Istio / Envoy / Kiali
    │
    └── monitoring namespace
        └── Prometheus / Grafana / Jaeger
```
<br>

## 11. EKS, EC2, Pod 관계

EKS와 EC2, Pod의 관계는 아래처럼 보면 됩니다.

```text
EKS Cluster
├── Control Plane
│   └── AWS가 관리
│
└── Worker Node
    ├── EC2 Instance 1
    │   ├── Pod
    │   ├── Pod
    │   └── Pod
    │
    ├── EC2 Instance 2
    │   ├── Pod
    │   ├── Pod
    │   └── Pod
    │
    └── EC2 Instance 3
        ├── Pod
        ├── Pod
        └── Pod
```

정리하면 아래와 같습니다.

```text
EKS = Kubernetes 클러스터
EC2 = 워커 노드
Pod = EC2 위에서 실행되는 컨테이너 단위
```
<br>

## 12. SSH와 kubectl 차이

| 구분 | 접근 대상 | 사용 목적 |
|---|---|---|
| SSH | EC2 워커 노드 | 서버 내부 직접 접근 |
| kubectl | EKS Kubernetes API Server | 클러스터와 Pod 관리 |

즉, SSH로 접근하는 대상은 EKS가 아닙니다.

```text
SSH 접속 대상 = EC2 워커 노드
kubectl 접근 대상 = EKS 클러스터
```
<br>

## 13. 한 줄 요약

```text
EKS는 AWS가 관리해주는 Kubernetes 클러스터이고,
EC2 워커 노드는 그 클러스터 안에서 실제 Pod를 실행하는 서버입니다.
EKS는 kubectl로 접근하고,
EC2 워커 노드는 필요할 때 SSH로 접근합니다.
```