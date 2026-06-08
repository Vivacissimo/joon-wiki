---
title: Istio란?
description: 
published: true
date: 2026-06-05T05:09:09.871Z
tags: 
editor: markdown
dateCreated: 2026-06-04T06:14:03.427Z
---

# Istio란?

**Istio**는 Kubernetes 환경에서 서비스 간 통신을 더 정교하게 제어하기 위한 **서비스 메시(Service Mesh) 도구**입니다.

Kubernetes가 Pod를 실행하고 Service로 연결해주는 역할을 한다면, Istio는 그 위에서 서비스 간 통신을 관리합니다.

예를 들어 다음과 같은 것들을 제어할 수 있습니다.

- 서비스 간 트래픽 분배
- Canary / Blue-Green 배포
- 서비스 간 통신 암호화
- 요청 실패 시 재시도
- 응답 시간 및 에러율 모니터링
- 장애 상황 테스트 및 복구 정책 적용

<br>

## 1. Istio를 왜 사용하는가?

마이크로서비스 구조에서는 하나의 서비스가 여러 서비스와 계속 통신합니다.

예를 들어 의료 서비스 프로젝트에서는 다음과 같은 흐름이 있을 수 있습니다.

```text
patient-service
  → appointment-service
  → prescription-service
  → notification-service
```

서비스가 많아질수록 다음과 같은 문제가 생깁니다.

- 어떤 서비스가 어떤 서비스를 호출하는지 파악하기 어렵다.
- 특정 서비스에서 에러가 발생했을 때 원인 추적이 어렵다.
- 신규 버전을 배포할 때 전체 트래픽을 바로 넘기기 부담스럽다.
- 서비스 간 통신을 암호화하고 싶은데 코드 수정이 많아진다.
- 장애 발생 시 자동 재시도나 차단 정책이 필요하다.

Istio는 이런 문제를 해결하기 위해 사용합니다.
<br>

## 2. Istio의 핵심 구조

Istio는 각 서비스 Pod 옆에 **Envoy Proxy**라는 사이드카 컨테이너를 붙여서 동작합니다.

```text
patient-service Pod
├── patient-service 컨테이너
└── Envoy sidecar
```

서비스가 다른 서비스로 요청을 보낼 때 직접 보내는 것이 아니라 Envoy를 거쳐서 통신합니다.

```text
patient-service
  → Envoy
  → Envoy
  → prescription-service
```

이 Envoy가 중간에서 다음 역할을 수행합니다.

- 트래픽 라우팅
- 서비스 간 인증 및 암호화
- 요청/응답 메트릭 수집
- 장애 대응 정책 적용
- 서비스 호출 흐름 추적
<br>

## 3. Istio의 주요 기능
<br>

### 3.1 트래픽 제어

Istio를 사용하면 서비스 버전별로 트래픽 비율을 나눌 수 있습니다.

예를 들어 `prescription-service`에 v1, v2 버전이 있을 때 다음처럼 배포할 수 있습니다.

```text
사용자 트래픽
├── 80% → prescription-service v1
└── 20% → prescription-service v2
```

이 방식이 **Canary 배포**입니다.

처음에는 신규 버전에 20%만 트래픽을 보내고, 문제가 없으면 50%, 100%로 점진적으로 늘릴 수 있습니다.

```text
1단계: v1 80% / v2 20%
2단계: v1 50% / v2 50%
3단계: v1 0%  / v2 100%
```

Istio에서는 주로 다음 리소스를 사용합니다.

| 리소스 | 역할 |
|---|---|
| `VirtualService` | 트래픽 라우팅 규칙 정의 |
| `DestinationRule` | 서비스 버전, 로드밸런싱, Circuit Breaker 설정 |
<br>

### 3.2 서비스 간 통신 보안

Istio는 서비스 간 통신에 **mTLS**를 적용할 수 있습니다.

mTLS는 서비스끼리 서로 인증하고 통신을 암호화하는 방식입니다.

```text
patient-service
  → 암호화된 통신
  → prescription-service
```

이를 통해 서비스 간 통신을 더 안전하게 만들 수 있습니다.

Istio에서는 주로 다음 리소스를 사용합니다.

| 리소스 | 역할 |
|---|---|
| `PeerAuthentication` | mTLS 정책 설정 |
| `AuthorizationPolicy` | 서비스 접근 권한 설정 |

예를 들어 Namespace 전체에 `mTLS STRICT` 모드를 적용하면, 해당 Namespace 안의 서비스들은 암호화된 통신만 허용하도록 구성할 수 있습니다.
<br>

### 3.3 모니터링과 관찰성

Istio를 사용하면 서비스 코드에 직접 많은 모니터링 로직을 넣지 않아도 서비스 간 통신 정보를 수집할 수 있습니다.

확인할 수 있는 정보는 다음과 같습니다.

- 어떤 서비스가 어떤 서비스를 호출하는지
- 요청 수가 얼마나 되는지
- 에러율이 몇 %인지
- 응답 시간이 얼마나 걸리는지
- 특정 요청이 어느 서비스에서 지연되는지

보통 Istio와 함께 다음 도구들을 사용합니다.

| 도구 | 역할 |
|---|---|
| Kiali | 서비스 간 트래픽 흐름 시각화 |
| Prometheus | 메트릭 수집 |
| Grafana | 대시보드 시각화 |
| Jaeger | 분산 트레이싱 |
<br>

### 3.4 장애 대응

Istio는 장애 상황에 대비하기 위한 여러 정책을 설정할 수 있습니다.

| 기능 | 설명 |
|---|---|
| Retry | 요청 실패 시 다시 요청 |
| Timeout | 응답이 너무 늦으면 요청 종료 |
| Circuit Breaker | 장애 서비스로 요청이 몰리는 것 방지 |
| Fault Injection | 일부러 장애나 지연을 넣어 테스트 |
| Outlier Detection | 문제가 있는 Pod를 일정 시간 제외 |

예를 들어 특정 서비스가 계속 5xx 에러를 반환하면 해당 Pod로 트래픽을 보내지 않도록 설정할 수 있습니다.

```text
정상 Pod → 트래픽 유지
문제 Pod → 일정 시간 제외
```
<br>

## 4. Istio에서 자주 쓰는 리소스

| 리소스 | 설명 |
|---|---|
| `Gateway` | 외부 트래픽을 클러스터 안으로 받는 진입점 |
| `VirtualService` | 트래픽 라우팅 규칙 정의 |
| `DestinationRule` | 서비스 버전, 로드밸런싱, Circuit Breaker 설정 |
| `PeerAuthentication` | mTLS 정책 설정 |
| `AuthorizationPolicy` | 서비스 접근 권한 설정 |
| `Telemetry` | 메트릭, 로그, 트레이싱 설정 |

---

## 5. 프로젝트에서 Istio를 쓰는 이유

프로젝트에서는 단순히 서비스를 배포하는 것을 넘어서, 운영 환경에 가까운 네트워크 제어와 모니터링을 구현해야 합니다.

Istio를 사용하면 다음 과업을 구현할 수 있습니다.

- 서비스 메시 도입
- Canary 배포
- Blue/Green 배포
- mTLS 기반 서비스 간 통신 암호화
- Kiali를 통한 서비스 토폴로지 확인
- Prometheus / Grafana 기반 Envoy 리소스 모니터링
- Jaeger 기반 분산 트레이싱
- Retry / Circuit Breaker 기반 장애 대응
- VirtualService 가중치 조정으로 빠른 롤백
<br>

## 6. 프로젝트 예시

의료 서비스 프로젝트에서 `prescription-service`에 Canary 배포를 적용한다고 하면 다음과 같이 구성할 수 있습니다.

```text
사용자 요청
  → Istio Gateway
  → VirtualService
  ├── 80% → prescription-service v1
  └── 20% → prescription-service v2
```

문제가 없다면 다음 단계로 전환합니다.

```text
1단계: v1 80% / v2 20%
2단계: v1 50% / v2 50%
3단계: v1 0%  / v2 100%
```

문제가 발생하면 바로 이전 버전으로 트래픽을 되돌릴 수 있습니다.

```text
v2 장애 발생
  → VirtualService 수정
  → v1 100%로 즉시 롤백
```
<br>

## 7. Kubernetes 기본 배포와 Istio 배포의 차이

| 구분 | Kubernetes 기본 배포 | Istio 적용 배포 |
|---|---|---|
| 트래픽 분배 | Service가 Pod로 단순 분산 | 버전별 가중치 분배 가능 |
| Canary 배포 | 별도 구현 필요 | VirtualService로 쉽게 구현 가능 |
| 서비스 간 암호화 | 별도 인증서/설정 필요 | mTLS 자동 적용 가능 |
| 장애 대응 | Liveness/Readiness 중심 | Retry, Timeout, Circuit Breaker 가능 |
| 모니터링 | 별도 계측 필요 | Envoy 기반 메트릭 수집 가능 |
| 서비스 흐름 확인 | 파악 어려움 | Kiali로 시각화 가능 |
<br>

## 8. Istio를 사용할 때 주의할 점

Istio는 기능이 강력하지만 그만큼 운영 복잡도가 올라갈 수 있습니다.

주의해야 할 점은 다음과 같습니다.

- Envoy sidecar가 추가되기 때문에 CPU/Memory 사용량이 증가한다.
- 서비스 메시 개념을 이해해야 하므로 학습 난이도가 있다.
- VirtualService, DestinationRule, Gateway 등 Istio 리소스를 별도로 관리해야 한다.
- 문제가 발생했을 때 Kubernetes 문제인지 Istio 설정 문제인지 구분해야 한다.
- 소규모 프로젝트에서는 과한 구성이 될 수 있다.

따라서 운영 환경에서는 Istio의 기능을 모두 쓰기보다는 프로젝트 목적에 맞게 필요한 기능부터 단계적으로 적용하는 것이 좋다.
<br>

## 9. 심화 프로젝트 적용 방향

이번 심화 프로젝트에서는 Istio를 다음 순서로 적용하는 것이 적절합니다.
<br>

### 9.1 Istio 설치

```text
Istio 설치
  → istio-system Namespace 생성
  → Istio control plane 구성
```

### 9.2 서비스 Namespace에 Sidecar 자동 주입 활성화

```bash
kubectl label namespace medical istio-injection=enabled
```

이 설정을 적용하면 `medical` Namespace에 새로 생성되는 Pod에는 Envoy sidecar가 자동으로 붙습니다.

### 9.3 Canary 배포 구성

`prescription-service` 또는 `appointment-service`에 v1, v2 버전을 만들고, `VirtualService`와 `DestinationRule`로 트래픽 비율을 조정합니다.

```text
v1 80% / v2 20%
  → v1 50% / v2 50%
  → v2 100%
```

### 9.4 Kiali로 트래픽 흐름 확인

Kiali를 통해 서비스 간 호출 관계를 확인합니다.

```text
patient-service
  → appointment-service
  → prescription-service
  → notification-service
```

### 9.5 Prometheus / Grafana로 Envoy 리소스 모니터링

Envoy sidecar의 CPU, Memory, Request 수, Error rate, Latency 등을 Prometheus로 수집하고 Grafana에서 시각화합니다.
<br>

### 9.6 장애 대응 정책 적용

Retry, Timeout, Circuit Breaker를 설정하여 장애 상황에서 서비스가 어떻게 동작하는지 검증합니다.
<br>

## 10. 한 줄 요약

**Istio는 Kubernetes에서 서비스 간 통신을 제어하고, 보안과 모니터링을 강화하며, Canary/Blue-Green 같은 무중단 배포를 가능하게 해주는 서비스 메시 도구입니다.**
<br>

## 참고 자료

- [Istio 공식 문서](https://istio.io/latest/docs/)
- [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
- [Istio Security](https://istio.io/latest/docs/concepts/security/)
- [Istio Observability](https://istio.io/latest/docs/concepts/observability/)
- [Kiali 공식 문서](https://kiali.io/docs/)
- [Envoy Proxy 공식 문서](https://www.envoyproxy.io/docs)