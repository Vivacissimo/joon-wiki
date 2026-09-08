---
title: Helm values.yaml
description: MoonCheap Helm values.yaml 주요 항목 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,helm,values,kubernetes
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# values.yaml

## 역할

Helm Template에서 사용하는 기본 설정값 모음.

```text
values.yaml
   +
templates/*.yaml
   ↓
Kubernetes Manifest
```

서비스별 Values가 공통 기본값을 덮어씀.

```text
values.yaml
   ↓
values/dev/backend.yaml
   ↓
Backend Manifest
```

---

## 주요 항목

### app

```yaml
app:
  name: app
  partOf: moongcheap
```

- 서비스 이름
- MoonCheap 프로젝트 소속 Label

### replicaCount

```yaml
replicaCount: 1
```

- 기본 Pod 개수
- HPA / KEDA 활성화 시 Autoscaler가 관리

### image

```yaml
image:
  repository: ""
  tag: "0000000"
  pullPolicy: IfNotPresent
```

- `repository`: ECR Image 주소
- `tag`: Image 버전
- `pullPolicy`: Image Pull 정책

MoonCheap Image Tag

```text
Git Commit SHA
```

예

```yaml
tag: a82f91c
```

### container

```yaml
container:
  port: 8080
  portName: http
  command: []
  args: []
```

- Container Port
- Port 이름
- 실행 Command / Argument override

서비스별 예

```text
Frontend → 3000
Backend  → 8080
AI       → 8000
```

실제 값은 Application 확인 후 확정.

### config

```yaml
config: {}
```

민감하지 않은 환경변수.

예

```yaml
config:
  SPRING_PROFILES_ACTIVE: dev
  AI_API_URL: http://moongcheap-ai-api:8000
```

### existingSecret

```yaml
existingSecret: ""
```

기존 Kubernetes Secret 이름 지정.

실제 Password / API Key는 Git에 저장하지 않음.

### service

```yaml
service:
  enabled: true
  type: ClusterIP
  port: 80
  targetPort: http
```

- Kubernetes Service 생성 여부
- 기본 내부 통신용 `ClusterIP`
- Service Port와 Container Port 연결

```text
Service :80
    ↓
Container :8080
```

### probes

```yaml
probes:
  readiness:
  liveness:
  startup:
```

Readiness

- 요청을 받을 준비가 됐는지 확인

Liveness

- Container가 정상 동작 중인지 확인
- 실패 지속 시 재시작

Startup

- 초기 기동 완료 여부 확인
- LLM처럼 시작이 느린 Application에 유용

### resources

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

`requests`

- Scheduler가 Node 배치 시 기준으로 사용하는 최소 요구량

`limits`

- Container 최대 사용량

현재 값은 기본 초깃값.

실제 부하 테스트 후 조정.

### nodeSelector / tolerations / affinity

```yaml
nodeSelector: {}
tolerations: []
affinity: {}
```

Pod 배치 제어.

AI LLM 예

```yaml
nodeSelector:
  workload: gpu
```

GPU 전용 Taint 사용 시 `tolerations` 추가.

### ingress

```yaml
ingress:
  enabled: false
```

외부 접근용 Ingress 설정.

- Frontend / Backend 외부 접근에 사용 가능
- AI API / LLM은 외부 비노출 방향

### autoscaling.hpa

```yaml
autoscaling:
  hpa:
    enabled: false
```

CPU / Memory 기반 Pod Autoscaling.

주요 값

- `minReplicas`
- `maxReplicas`
- `cpuUtilization`
- `memoryUtilization`

### autoscaling.keda

```yaml
autoscaling:
  keda:
    enabled: false
```

외부 Metric 기반 Autoscaling.

AI LLM Scale-to-Zero 후보.

```text
요청 없음
  ↓
LLM Pod 0
  ↓
GPU Node 제거
```

실제 Metric / Cold Start 측정 후 활성화.

### pdb

```yaml
pdb:
  enabled: false
```

Node Drain 등의 상황에서 Pod가 동시에 과도하게 종료되는 것을 방지.

### monitoring.serviceMonitor

```yaml
monitoring:
  serviceMonitor:
    enabled: false
```

Prometheus가 Application Metric을 수집하도록 연결.

예

```text
Backend → /actuator/prometheus
AI      → /metrics
```

### networkPolicy

```yaml
networkPolicy:
  enabled: false
```

Pod 간 통신 허용 범위 제어.

예

```text
Backend → AI API   O
Internet → AI API  X
```

세부 정책 합의 후 활성화.

---

## 우선 이해할 항목

```text
app
replicaCount
image
container
service
resources
```

기본 배포 흐름 이해 후 추가 확인

```text
Probe
Ingress
HPA / KEDA
PDB
ServiceMonitor
NetworkPolicy
```
