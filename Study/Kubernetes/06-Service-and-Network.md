---
title: 06. Service and Network
description: Service와 Kubernetes 네트워크
published: true
tags: kubernetes
editor: markdown
---

# Service와 Network

Pod는 재생성될 수 있고 IP도 바뀔 수 있음.

고정된 접근 지점을 만들기 위해 `Service` 사용.

```text
Client
  ↓
Service
  ↓
Pod A / Pod B / Pod C
```

## Service

Label Selector로 대상 Pod를 찾고 트래픽 전달.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

## Service 종류

### ClusterIP

Cluster 내부에서만 접근.

기본 Service 타입.

### NodePort

각 Node의 특정 Port를 외부에 개방.

```text
<NodeIP>:<NodePort>
```

실습 환경에서 자주 사용.

### LoadBalancer

Cloud Load Balancer와 연결.

AWS EKS 같은 환경에서 외부 서비스 노출에 사용.

### ExternalName

외부 DNS 이름을 Kubernetes Service처럼 사용.

## Pod Network

모든 Pod가 서로 통신할 수 있도록 구성된 내부 네트워크.

Kubernetes 자체가 실제 네트워크를 만들지는 않음.

CNI 플러그인 필요.

대표 CNI:

- Calico
- Cilium
- Flannel

## CNI

Container Network Interface.

주요 역할:

- Pod IP 할당
- Pod 간 통신
- Node 간 Pod 통신
- NetworkPolicy 지원 여부에 따른 정책 적용

## Service Network와 Pod Network

```text
Node Network    → 실제 서버 IP
Pod Network     → Pod IP
Service Network → Service ClusterIP
```

서로 다른 CIDR 사용 권장.

## DNS

CoreDNS가 Cluster 내부 DNS 담당.

같은 Namespace:

```text
http://backend
```

다른 Namespace:

```text
http://backend.prod
```

전체 FQDN:

```text
backend.prod.svc.cluster.local
```

## 다음

`07-Config-and-Storage`에서 설정값과 데이터 저장 방식 정리.
