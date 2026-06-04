---
title: Monitoring
description: 
published: true
date: 2026-06-04T08:22:13.453Z
tags: 
editor: markdown
dateCreated: 2026-06-04T07:48:45.722Z
---

# 모니터링 및 관찰성 스택

## 개요

모니터링 및 관찰성 도구를 정리

Istio 기반 서비스 메시 환경에서 서비스 간 트래픽 흐름, 에러율, 응답 시간, Envoy Sidecar의 CPU/Memory 사용량, 분산 트레이싱 정보를 확인하는 것을 주로 작성했다. 

---

## 사용하는 모니터링 도구 목록

| 구분 | 도구 | 사용 여부 | 역할 
|---|---|---|---|---|
| 서비스 메시 시각화 | [Kiali](/Monitoring/kiali/what-is-kiali) | 필수 | 서비스 간 트래픽 흐름, 에러율, 서비스 토폴로지 시각화 
| 메트릭 수집 | [Prometheus](./Prometheus/what-is-prometheus) | 필수 | Istio, Envoy, Kubernetes 메트릭 수집 
| 대시보드 | [Grafana](./Grafana/what-is-grafana) | 필수 | Prometheus 메트릭 기반 대시보드 시각화 
| 분산 트레이싱 | [Jaeger](./Jaeger/what-is-jaeger) | 선택 | 서비스 간 요청 흐름 추적 및 병목 구간 분석
| 서비스 메시 메트릭 | [Envoy Metrics](./Envoy/what-is-envoy-metrics) | 필수 측정 대상 | Envoy Sidecar의 요청 수, 에러율, 응답 시간, CPU/Memory 확인 

---

## 필수 구성

### Kiali

Kiali는 Istio 서비스 메시의 트래픽 흐름을 시각화하는 도구

서비스 간 호출 관계, 트래픽 흐름, 에러율, 응답 시간, mTLS 적용 여부를 확인하는 데 사용

페이지: [Kiali](/Monitoring/kiali/what-is-kiali)  
<br>

### Prometheus

Prometheus는 Kubernetes와 Istio 환경에서 발생하는 메트릭을 수집하는 도구

본 프로젝트에서는 Istio와 Envoy에서 발생하는 요청 수, 에러율, 응답 시간, Envoy Sidecar CPU/Memory 사용량을 수집

페이지: [Prometheus](./Prometheus/what-is-prometheus)\
<br>

### Grafana

Grafana는 Prometheus에서 수집한 메트릭을 대시보드로 시각화하는 도구이다.

서비스별 요청 수, 에러율, 응답 시간, P95/P99 지연 시간, Envoy CPU/Memory 사용량을 확인하는 데 사용한다.

페이지: [Grafana](./Grafana/what-is-grafana)  
주소: `./Grafana/what-is-grafana`
<br>

### Envoy Metrics

Envoy Metrics는 Istio Sidecar Proxy인 Envoy에서 발생하는 메트릭이다.

Envoy는 별도의 대시보드 도구라기보다는 Prometheus가 수집하고 Grafana에서 시각화하는 측정 대상이다.

확인 항목은 다음과 같다.

- 서비스 간 요청 수
- HTTP 응답 코드
- 에러율
- 응답 시간
- Retry 발생 여부
- Circuit Breaker 동작 여부
- Envoy Sidecar CPU 사용량
- Envoy Sidecar Memory 사용량

페이지: [Envoy Metrics](./Envoy/what-is-envoy-metrics)  
주소: `./Envoy/what-is-envoy-metrics`
<br>

## 선택 구성
<br>

### Jaeger

Jaeger는 분산 트레이싱 도구이다.

하나의 요청이 여러 서비스를 거쳐 처리될 때, 각 서비스에서 걸린 시간과 병목 구간을 추적하는 데 사용한다.

예를 들어 다음과 같은 요청 흐름을 추적할 수 있다.

```text
Appointment Service
  ↓
Prescription Service
  ↓
Notification Service