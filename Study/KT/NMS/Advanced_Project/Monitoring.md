---
title: Prometheus&Grafana
description: 
published: true
date: 2026-06-25T06:49:25.920Z
tags: 
editor: markdown
dateCreated: 2026-06-25T06:49:25.920Z
---

## Prometheus 및 Grafana 모니터링 구축

### 페이지 목적

Prometheus와 Grafana를 이용해 Kubernetes 및 Istio 메트릭을 수집하고 시각화한 과정을 정리한다.

### 포함할 내용

- 모니터링 구성 목적
- Prometheus 역할
- Grafana 역할
- `kube-prometheus-stack` 설치
- Prometheus Target 확인
- Grafana Data Source 연결
- Grafana 접속 방법
- 구성한 대시보드
- 각 패널의 PromQL
- 단위와 Legend 설정
- 대시보드 새로고침 설정
- 모니터링 결과

### 구성한 주요 패널

- Envoy CPU 사용량
- Envoy 메모리 사용량
- 요청 처리량
- P95 응답 시간
- 요청 성공률
- 5xx 오류율

### 페이지에 들어갈 주요 명령어

```powershell
kubectl get pods -n monitoring

kubectl port-forward `
  -n monitoring `
  svc/kube-prometheus-stack-grafana `
  3000:80
```