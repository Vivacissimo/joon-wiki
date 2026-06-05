---
title: Helm으로 설치하기
description: 
published: true
date: 2026-06-05T02:33:36.255Z
tags: 
editor: markdown
dateCreated: 2026-06-05T02:33:36.255Z
---

# 모니터링 툴 Helm 설치 방법

심화 프로젝트의 모니터링 구성은 다음 순서로 설치한다.

```text
1. Helm repo 추가
2. Prometheus + Grafana 설치
3. Istio / Envoy 메트릭 수집 설정
4. Kiali 설치
5. Jaeger 설치
6. Kiali와 Jaeger 연결
7. 전체 설치 확인
```

---

## 0. Helm repo 추가

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add kiali https://kiali.org/helm-charts
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts

helm repo update
```

Prometheus Community는 `prometheus-community` Helm repo를 제공하고, Grafana도 공식 Helm repo를 제공한다.  
Kiali는 Helm 설치 시 `kiali-operator` 방식으로 설치한다.

---

## 1. Prometheus + Grafana 설치

Prometheus와 Grafana를 각각 따로 설치하지 않고, `kube-prometheus-stack`으로 설치한다.

`kube-prometheus-stack`에는 다음 구성 요소가 포함된다.

```text
Prometheus
Grafana
Alertmanager
Prometheus Operator
ServiceMonitor / PodMonitor CRD
```

### Namespace 생성

```bash
kubectl create namespace monitoring
```

### kube-prometheus-stack 설치

```bash
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --set grafana.enabled=true \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false
```

### 설치 확인

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

---

## 2. Grafana 접속

### Port-forward 실행

```bash
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```

### 브라우저 접속

```text
http://localhost:3000
```

### Grafana admin 비밀번호 확인

```bash
kubectl -n monitoring get secret kube-prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d
echo
```

### 기본 로그인 정보

```text
ID: admin
PW: 위 명령어 결과
```

---

## 3. Istio / Envoy 메트릭 수집 설정

Prometheus가 Istio 사이드카인 Envoy 메트릭을 수집해야 Kiali와 Grafana에서 서비스 메시 메트릭을 확인할 수 있다.

Envoy 메트릭 수집을 위해 `PodMonitor`를 생성한다.

---

### istio-podmonitor.yaml 생성

```bash
vi istio-podmonitor.yaml
```

### 내용 작성

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: istio-envoy
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  namespaceSelector:
    any: true
  selector:
    matchExpressions:
      - key: security.istio.io/tlsMode
        operator: Exists
  podMetricsEndpoints:
    - path: /stats/prometheus
      port: http-envoy-prom
      interval: 15s
```

### 적용

```bash
kubectl apply -f istio-podmonitor.yaml
```

---

## 4. Istio Control Plane 메트릭 수집 설정

Istio control plane인 `istiod` 메트릭도 Prometheus에서 수집할 수 있도록 `ServiceMonitor`를 생성한다.

---

### istiod-servicemonitor.yaml 생성

```bash
vi istiod-servicemonitor.yaml
```

### 내용 작성

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: istiod
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  namespaceSelector:
    matchNames:
      - istio-system
  selector:
    matchLabels:
      app: istiod
  endpoints:
    - port: http-monitoring
      interval: 15s
```

### 적용

```bash
kubectl apply -f istiod-servicemonitor.yaml
```

### 확인

```bash
kubectl get podmonitor -n monitoring
kubectl get servicemonitor -n monitoring
```

---

## 5. Kiali 설치

Kiali는 Istio 서비스 메시를 시각화하는 콘솔이다.

Kiali를 통해 다음 항목을 확인할 수 있다.

```text
서비스 간 트래픽 흐름
서비스별 에러율
서비스별 응답 상태
mTLS 적용 상태
Istio 설정 상태
```

Kiali는 Prometheus 데이터를 사용해서 토폴로지, 에러율, 트래픽 흐름을 보여준다.

---

### Kiali Operator 설치

```bash
helm upgrade --install kiali-operator kiali/kiali-operator \
  -n kiali-operator \
  --create-namespace \
  --set cr.create=true \
  --set cr.namespace=istio-system \
  --set cr.spec.auth.strategy=anonymous \
  --set cr.spec.external_services.prometheus.url="http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090" \
  --set cr.spec.external_services.grafana.enabled=true \
  --set cr.spec.external_services.grafana.internal_url="http://kube-prometheus-stack-grafana.monitoring.svc.cluster.local"
```

### 설치 확인

```bash
kubectl get pods -n kiali-operator
kubectl get pods -n istio-system | grep kiali
kubectl get svc -n istio-system | grep kiali
```

---

## 6. Kiali 접속

### Port-forward 실행

```bash
kubectl -n istio-system port-forward svc/kiali 20001:20001
```

### 브라우저 접속

```text
http://localhost:20001
```

### Kiali에서 확인할 메뉴

```text
Graph
Applications
Workloads
Services
Istio Config
```

---

## 7. Jaeger 설치

Jaeger는 선택 과업이다.

Prometheus, Grafana, Kiali까지 먼저 설치한 뒤, 분산 트레이싱이 필요할 때 Jaeger를 추가한다.

Jaeger는 요청 하나가 여러 서비스를 거쳐 갈 때 어느 서비스에서 지연이 발생했는지 확인하는 분산 트레이싱 도구이다.

예시:

```text
patient-service → prescription-service → notification-service
```

이 흐름에서 특정 요청이 어디서 느려졌는지 확인할 수 있다.

---

### Namespace 생성

```bash
kubectl create namespace tracing
```

### Jaeger 설치

```bash
helm upgrade --install jaeger jaegertracing/jaeger \
  -n tracing
```

### 설치 확인

```bash
kubectl get pods -n tracing
kubectl get svc -n tracing
```

---

## 8. Jaeger UI 접속

Jaeger UI 접속용 서비스명을 확인한다.

```bash
kubectl get svc -n tracing
```

서비스 이름이 `jaeger-query`이면 다음 명령어를 실행한다.

```bash
kubectl -n tracing port-forward svc/jaeger-query 16686:16686
```

브라우저에서 접속한다.

```text
http://localhost:16686
```

---

## 9. Kiali에 Jaeger 연결

Kiali에서 Jaeger Trace를 보려면 Kiali CR에 tracing 정보를 추가해야 한다.

---

### Kiali CR 수정

```bash
kubectl edit kiali kiali -n istio-system
```

### 아래 내용을 `spec` 아래에 추가

기존에 `external_services`가 있다면 새로 만들지 말고 기존 항목에 병합한다.

```yaml
spec:
  external_services:
    tracing:
      enabled: true
      provider: jaeger
      use_grpc: false
      internal_url: "http://jaeger-query.tracing.svc.cluster.local:16686"
```

### Kiali 재시작

```bash
kubectl rollout restart deployment/kiali -n istio-system
```

### 확인

```bash
kubectl get pods -n istio-system | grep kiali
```

---

## 10. 전체 설치 확인

### Pod 상태 확인

```bash
kubectl get pods -n monitoring
kubectl get pods -n istio-system
kubectl get pods -n kiali-operator
kubectl get pods -n tracing
```

### Service 상태 확인

```bash
kubectl get svc -n monitoring
kubectl get svc -n istio-system
kubectl get svc -n tracing
```

### Helm release 확인

```bash
helm list -A
```


## 11. 각 도구 역할 정리

| 도구 | 역할 |
|---|---|
| Prometheus | Kubernetes, Istio, Envoy 메트릭 수집 |
| Grafana | Prometheus 메트릭 시각화 |
| Kiali | Istio 서비스 메시 토폴로지 시각화 |
| Jaeger | 서비스 간 요청 흐름 추적 |
| PodMonitor | Pod 단위 메트릭 수집 대상 정의 |
| ServiceMonitor | Service 단위 메트릭 수집 대상 정의 |
