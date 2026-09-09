---
title: Grafana Install
description: kube-prometheus-stack 기반 Grafana 설치
published: true
date: 2026-06-24T09:08:59.576Z
tags: monitoring, grafana, prometheus, kubernetes
editor: markdown
dateCreated: 2026-06-17T03:17:26.258Z
---

# Grafana Install

## 1. Monitoring Namespace

Prometheus, Grafana 설치용 Namespace.

```bash
kubectl create namespace monitoring
```

이미 있으면 create 에러. 그대로 넘어가도 됨.

확인

```bash
kubectl get ns
```

---

## 2. Prometheus Community Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 3. kube-prometheus-stack 설치

한 번에 설치되는 구성

- Prometheus
- Grafana
- Alertmanager
- node-exporter
- kube-state-metrics
- Prometheus Operator

설치

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring
```

출력 예시

```text
NAME: kube-prometheus-stack
LAST DEPLOYED: Wed Jun 17 02:18:15 2026
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

---

## 4. Pod 상태

```bash
kubectl get pods -n monitoring
```

정상 상태 예시

```text
NAME                                                        READY   STATUS    RESTARTS   AGE
alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   0          ...
kube-prometheus-stack-grafana-xxxxx                         3/3     Running   0          ...
kube-prometheus-stack-kube-state-metrics-xxxxx              1/1     Running   0          ...
kube-prometheus-stack-operator-xxxxx                        1/1     Running   0          ...
kube-prometheus-stack-prometheus-node-exporter-xxxxx        1/1     Running   0          ...
prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          ...
```

처음엔 Grafana가 `ContainerCreating`일 수 있음.

```text
kube-prometheus-stack-grafana-xxxxx   0/3   ContainerCreating
```

조금 기다린 뒤 다시 확인.

```bash
kubectl get pods -n monitoring
```

계속 그대로면 describe.

```bash
kubectl describe pod -n monitoring -l app.kubernetes.io/name=grafana
```

---

## 5. Service 상태

```bash
kubectl get svc -n monitoring
```

예시

```text
NAME                                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)
alertmanager-operated                            ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP
kube-prometheus-stack-alertmanager               ClusterIP   ...              <none>        9093/TCP,8080/TCP
kube-prometheus-stack-grafana                    ClusterIP   ...              <none>        80/TCP
kube-prometheus-stack-kube-state-metrics         ClusterIP   ...              <none>        8080/TCP
kube-prometheus-stack-operator                   ClusterIP   ...              <none>        443/TCP
kube-prometheus-stack-prometheus                 ClusterIP   ...              <none>        9090/TCP,8080/TCP
kube-prometheus-stack-prometheus-node-exporter   ClusterIP   ...              <none>        9100/TCP
prometheus-operated                              ClusterIP   None             <none>        9090/TCP
```

Grafana Service = `80/TCP`.

---

## 6. Grafana Port Forward

외부 서버에서 접속할 때 `--address 0.0.0.0` 사용.

```bash
kubectl port-forward --address 0.0.0.0 -n monitoring svc/kube-prometheus-stack-grafana 3000:80
```

출력

```text
Forwarding from 0.0.0.0:3000 -> 3000
```

접속 흐름

```text
PC Browser
   ↓
외부 TCP 주소
   ↓
Master Node :3000
   ↓
kubectl port-forward
   ↓
Grafana Service :80
```

---

## 7. Grafana 로그인

ID

```text
admin
```

비밀번호 확인

```bash
kubectl --namespace monitoring get secrets kube-prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

---

## 8. Port Forward 백그라운드 실행

터미널을 닫아도 유지하려면 `nohup`.

기존 port-forward가 있으면 먼저 종료.

```bash
nohup kubectl port-forward --address 0.0.0.0 -n monitoring svc/kube-prometheus-stack-grafana 3000:80 > grafana-portforward.log 2>&1 &
```

---

## 9. 프로세스 / 포트 확인

프로세스

```bash
ps -ef | grep port-forward
```

포트

```bash
ss -lntp | grep 3000
```

로그

```bash
tail -f grafana-portforward.log
```

---

## 10. Port Forward 종료

PID 확인

```bash
ps -ef | grep port-forward
```

종료

```bash
kill PID번호
```

강제 종료가 필요할 때만

```bash
kill -9 PID번호
```

---

## 11. Prometheus Port Forward

Prometheus 포트 = `9090`.

```bash
nohup kubectl port-forward --address 0.0.0.0 -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090 > prometheus-portforward.log 2>&1 &
```

확인

```bash
ss -lntp | grep 9090
```

```bash
tail -f prometheus-portforward.log
```

---

## 12. Kiali Port Forward

Kiali 기본 포트 = `20001`.

```bash
nohup kubectl port-forward --address 0.0.0.0 -n istio-system svc/kiali 20001:20001 > kiali-portforward.log 2>&1 &
```

프로세스 확인

```bash
ps -ef | grep port-forward
```

종료

```bash
kill PID번호
```

---

## 같이 보기

- [Monitoring](../../Monitoring)
- [Kiali](../kiali/what-is-kiali)
