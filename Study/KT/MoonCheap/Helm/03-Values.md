---
title: Helm values.yaml
description: MoonCheap Helm values.yaml 항목과 Kubernetes 개념 정리
published: true
date: 2026-09-08T08:05:00.000Z
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

`values.yaml`에는 실제 Kubernetes Resource를 직접 정의하지 않음.

Template에서 사용할 값을 정의.

예

```yaml
replicaCount: 2
```

`deployment.yaml`에서 해당 값을 사용.

```yaml
replicas: {{ .Values.replicaCount }}
```

최종 Manifest

```yaml
replicas: 2
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

## Values에서 자주 보이는 표현

### `{}`

빈 Object / Map.

```yaml
podAnnotations: {}
```

현재 설정값 없음.

필요하면 Key / Value 형태로 추가.

```yaml
podAnnotations:
  example.com/test: "true"
```

### `[]`

빈 List.

```yaml
tolerations: []
```

현재 등록된 항목 없음.

필요하면 여러 항목 추가 가능.

### `""`

빈 문자열.

```yaml
existingSecret: ""
```

현재 지정된 이름 없음.

### `null`

값을 사용하지 않음.

```yaml
memoryUtilization: null
```

Memory 기준 HPA 설정을 사용하지 않는 상태.

---

# 기본 Application 설정

## app

```yaml
app:
  name: app
  partOf: moongcheap
```

### `name`

서비스 이름.

예

```text
frontend
backend
ai-api
ai-llm
```

Deployment / Service / Label 이름 생성에 사용.

### `partOf`

해당 서비스가 어떤 상위 Application에 속하는지 표시.

MoonCheap의 여러 서비스에 공통 Label 적용.

```text
frontend ─┐
backend  ─┼─ moongcheap
ai-api   ─┤
ai-llm   ─┘
```

---

## replicaCount

```yaml
replicaCount: 1
```

기본 Pod 개수.

```text
replicaCount: 1
→ Pod 1개

replicaCount: 3
→ Pod 3개
```

HPA / KEDA 활성화 시 Pod 개수는 Autoscaler가 관리.

---

# Container Image

## image

```yaml
image:
  repository: ""
  tag: "0000000"
  pullPolicy: IfNotPresent
```

### `repository`

Container Image 저장 위치.

MoonCheap에서는 ECR 사용 예정.

예

```text
123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/moongcheap-backend
```

### `tag`

Image 버전.

MoonCheap에서는 Git Commit SHA 사용.

```yaml
tag: a82f91c
```

최종 Image

```text
moongcheap-backend:a82f91c
```

### `pullPolicy`

Node가 Image를 언제 가져올지 결정.

`IfNotPresent`

- Node에 Image가 없으면 Pull
- 이미 있으면 기존 Image 사용

대표 값

```text
Always       → 실행할 때마다 Registry 확인
IfNotPresent → Node에 없을 때만 Pull
Never        → Registry에서 Pull하지 않음
```

---

## imagePullSecrets

```yaml
imagePullSecrets: []
```

Private Container Registry에서 Image를 Pull할 때 사용할 인증 정보.

예

```text
Pod
 ↓ Image Pull 요청
Private Registry
 ↓
인증 필요
```

이 인증 정보를 Kubernetes Secret으로 만들어 사용 가능.

```yaml
imagePullSecrets:
  - name: registry-secret
```

```text
Pod
 ↓
imagePullSecret
 ↓
Private Registry 인증
 ↓
Image Pull
```

### MoonCheap에서는

ECR을 사용하므로 일반적인 Docker Registry Secret을 직접 넣는 구조가 반드시 필요한 것은 아님.

EKS Node / IAM 권한을 통해 ECR Image Pull 권한을 제공하는 방식 사용 가능.

현재는 비어 있는 상태.

```yaml
imagePullSecrets: []
```

---

# Kubernetes Identity / Metadata

## serviceAccount

```yaml
serviceAccount:
  create: false
  name: ""
  annotations: {}
```

Pod가 Kubernetes 안에서 사용하는 **Identity**.

쉽게 보면

```text
사람
→ 사용자 계정

Pod
→ ServiceAccount
```

Pod가 Kubernetes API 또는 외부 Cloud Resource에 접근할 때 권한 기준으로 사용 가능.

```text
Backend Pod
    ↓
ServiceAccount
    ↓
권한 확인
    ↓
Kubernetes API / AWS Resource
```

### `create`

Helm Chart가 ServiceAccount를 새로 만들지 여부.

```yaml
create: true
```

Helm이 ServiceAccount 생성.

```yaml
create: false
```

새로 만들지 않음.

### `name`

사용할 ServiceAccount 이름.

```yaml
name: moongcheap-backend
```

### `annotations`

ServiceAccount에 추가 Metadata 설정.

EKS에서 AWS IAM Role 연결 시 사용할 수 있음.

예시 개념

```text
Backend Pod
   ↓
ServiceAccount
   ↓
IAM Role
   ↓
AWS Secrets Manager
```

MoonCheap에서는 워크로드별 최소 권한이 필요할 경우 ServiceAccount 분리를 고려.

---

## podAnnotations

```yaml
podAnnotations: {}
```

Pod에 추가하는 Annotation.

Annotation은 Kubernetes Resource에 붙이는 **부가 정보**.

Label처럼 Resource를 설명하지만 Selector 용도보다는 Tool / Controller에 추가 정보를 전달할 때 많이 사용.

예

```yaml
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
```

개념

```text
Pod
 ├─ 실제 실행 정보
 └─ Annotation
      └─ 다른 Tool이 참고할 추가 정보
```

현재 MoonCheap Chart에서는 별도 Annotation 없음.

---

## podLabels

```yaml
podLabels: {}
```

Pod에 추가 Label을 붙이는 설정.

Label은 Kubernetes Resource를 **구분 / 검색 / 선택**하기 위한 값.

예

```yaml
podLabels:
  tier: backend
  environment: dev
```

Service, NetworkPolicy 등이 Label을 기준으로 Pod를 선택 가능.

```text
Service
  ↓ selector
label: app=backend
  ↓
Backend Pod
```

### Label과 Annotation 차이

```text
Label
→ Resource 분류 / 선택에 사용
→ Service Selector, NetworkPolicy 등에 활용

Annotation
→ Resource에 추가 정보 기록
→ 외부 Controller / Tool 설정 등에 활용
```

---

# Security Context

## podSecurityContext

```yaml
podSecurityContext: {}
```

Pod 전체에 적용하는 보안 설정.

Pod 안의 여러 Container가 공통으로 사용할 Security 설정에 사용.

예

```yaml
podSecurityContext:
  fsGroup: 2000
```

Volume 파일 권한 등 Pod 단위 Security 설정에 사용.

---

## securityContext

```yaml
securityContext: {}
```

개별 Container에 적용하는 보안 설정.

예

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
```

개념

```text
Pod
├─ podSecurityContext
│    └─ Pod 전체 설정
│
└─ Container
     └─ securityContext
          └─ Container 개별 설정
```

대표 용도

- Root 사용자 실행 방지
- 특정 UID로 실행
- Privilege Escalation 방지
- Linux Capability 제어

현재 MoonCheap에서는 세부 Security 정책 확정 전이라 빈 값.

---

# Container 실행 설정

## container

```yaml
container:
  port: 8080
  portName: http
  command: []
  args: []
```

### `port`

Container Application이 Listen하는 Port.

서비스별 예상 값

```text
Frontend → 3000
Backend  → 8080
AI       → 8000
```

실제 Application 확인 후 확정.

### `portName`

Port에 붙이는 이름.

```yaml
portName: http
```

다른 Kubernetes Resource에서 숫자 대신 이름으로 참조 가능.

### `command`

Container 실행 Command override.

비어 있으면 Dockerfile의 `ENTRYPOINT` 사용.

### `args`

Container 실행 Argument override.

비어 있으면 Dockerfile의 기본 실행 설정 사용.

---

# Environment Variable / Secret

## config

```yaml
config: {}
```

민감하지 않은 환경변수.

ConfigMap으로 생성 후 Pod에 주입.

예

```yaml
config:
  SPRING_PROFILES_ACTIVE: dev
  AI_API_URL: http://moongcheap-ai-api:8000
```

```text
values
 ↓
ConfigMap
 ↓
Pod Environment Variable
```

Password / Token / API Key 저장 금지.

---

## existingSecret

```yaml
existingSecret: ""
```

이미 존재하는 Kubernetes Secret 이름 지정.

예

```yaml
existingSecret: moongcheap-backend-secret
```

```text
Secret
  ↓
Backend Pod
  ↓
DB_PASSWORD 등의 환경변수
```

실제 Password / API Key는 Git에 저장하지 않음.

---

# Service

## service

```yaml
service:
  enabled: true
  type: ClusterIP
  port: 80
  targetPort: http
  annotations: {}
```

Pod에 접근할 수 있는 고정 Endpoint 제공.

Pod IP는 생성 / 재시작에 따라 변경 가능.

그래서 다른 서비스가 Pod IP를 직접 호출하지 않고 Service를 호출.

```text
Backend
  ↓
moongcheap-ai-api Service
  ↓
AI API Pod
```

### `enabled`

Service 생성 여부.

### `type`

Service 종류.

현재 기본값

```yaml
type: ClusterIP
```

클러스터 내부 접근용.

### `port`

Service가 받는 Port.

```yaml
port: 80
```

### `targetPort`

Service가 전달할 Container Port.

```yaml
targetPort: http
```

`container.portName: http`를 참조.

```text
Service :80
    ↓
Container :8080
```

### `annotations`

Service에 추가 Metadata 설정.

Load Balancer Controller 등 외부 Controller가 참고하는 설정을 넣을 수 있음.

---

# Probe

## probes

```yaml
probes:
  readiness:
  liveness:
  startup:
```

### Readiness Probe

요청을 받을 준비가 됐는지 확인.

실패하면 Pod 자체를 종료하지 않음.

Service Endpoint에서 제외.

```text
Readiness 실패
 ↓
Pod 실행 중
 ↓
하지만 Traffic 전달 X
```

### Liveness Probe

Container가 정상 동작 중인지 확인.

지속 실패 시 Container 재시작.

### Startup Probe

Application 초기 기동 완료 여부 확인.

LLM처럼 기동 시간이 오래 걸리는 Application에 유용.

---

# Resource

## resources

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

### `requests`

Pod가 필요하다고 요청하는 Resource.

Scheduler가 어느 Node에 배치할지 판단할 때 사용.

```text
Pod requests
    ↓
Scheduler
    ↓
수용 가능한 Node 선택
```

### `limits`

Container가 사용할 수 있는 최대 Resource.

현재 값은 기본 초깃값.

실제 부하 테스트 후 조정.

---

# Pod 배치

## nodeSelector

```yaml
nodeSelector: {}
```

특정 Label을 가진 Node에 Pod를 배치.

AI LLM 예

```yaml
nodeSelector:
  workload: gpu
```

```text
AI LLM Pod
   ↓
workload=gpu Node 검색
   ↓
GPU Node 배치
```

---

## tolerations

```yaml
tolerations: []
```

Taint가 설정된 Node에 Pod가 들어갈 수 있도록 허용.

개념

```text
Taint
→ 이 Node에는 아무 Pod나 들어오지 못하게 제한

Toleration
→ 해당 제한을 허용받은 Pod
```

GPU Node 격리에 사용 가능.

```text
GPU Node
Taint: dedicated=gpu

AI LLM
Toleration: dedicated=gpu
```

---

## affinity

```yaml
affinity: {}
```

Pod를 어떤 Node 또는 다른 Pod 근처에 배치할지 정하는 고급 규칙.

대표 개념

```text
Node Affinity
→ 특정 조건의 Node 선호 / 강제

Pod Affinity
→ 특정 Pod와 가까이 배치

Pod Anti-Affinity
→ 특정 Pod와 떨어뜨려 배치
```

---

## topologySpreadConstraints

```yaml
topologySpreadConstraints: []
```

Pod가 특정 Node / AZ에 몰리지 않도록 분산 배치하는 규칙.

예

```text
Backend Pod 3개

Node A → 1
Node B → 1
Node C → 1
```

가용성 향상에 사용.

---

## terminationGracePeriodSeconds

```yaml
terminationGracePeriodSeconds: 30
```

Pod 종료 요청 후 강제 종료까지 기다리는 시간.

```text
SIGTERM 전달
   ↓
Application 정상 종료 대기
   ↓
30초
   ↓
종료되지 않으면 강제 종료
```

Connection 정리 / 작업 종료 시간을 확보하기 위한 값.

---

# Ingress

## ingress

```yaml
ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts: []
  tls: []
```

외부 요청을 Kubernetes Service로 전달하는 설정.

```text
Internet
   ↓
ALB / Ingress Controller
   ↓
Ingress
   ↓
Service
   ↓
Pod
```

### `className`

어떤 Ingress Controller가 처리할지 지정.

EKS + AWS Load Balancer Controller 사용 시 `alb` 사용 가능.

### `annotations`

ALB Scheme, Target Type 등 Controller용 추가 설정.

### `hosts`

접속 Domain.

### `tls`

HTTPS 관련 설정.

MoonCheap에서는 Frontend / Backend 외부 접근에 사용 가능.

AI API / LLM은 외부 비노출 방향.

---

# Autoscaling

## autoscaling.hpa

```yaml
autoscaling:
  hpa:
    enabled: false
    minReplicas: 1
    maxReplicas: 3
    cpuUtilization: 70
    memoryUtilization: null
```

CPU / Memory 사용률 기반 Pod Autoscaling.

```text
CPU 증가
  ↓
HPA
  ↓
Pod 증가
```

### 주요 값

`minReplicas`

- 최소 Pod 개수

`maxReplicas`

- 최대 Pod 개수

`cpuUtilization`

- 목표 평균 CPU 사용률

`memoryUtilization`

- 목표 평균 Memory 사용률
- `null`이면 사용하지 않음

---

## autoscaling.keda

```yaml
autoscaling:
  keda:
    enabled: false
    pollingInterval: 30
    cooldownPeriod: 300
    minReplicaCount: 0
    maxReplicaCount: 1
    triggers: []
```

외부 Metric / Event 기반 Autoscaling.

HPA보다 다양한 기준 사용 가능.

예

- Prometheus Metric
- Queue 길이
- Kafka Lag
- AWS SQS

MoonCheap에서는 AI LLM Scale-to-Zero 후보.

```text
요청 없음
  ↓
KEDA
  ↓
LLM Pod 0
  ↓
Karpenter
  ↓
GPU Node 제거
```

실제 Metric / Cold Start 측정 후 활성화.

---

# 가용성

## pdb

```yaml
pdb:
  enabled: false
  minAvailable: 1
  maxUnavailable: null
```

PodDisruptionBudget.

Node Drain / Node Upgrade 같은 **계획된 중단** 상황에서 Pod가 너무 많이 동시에 종료되지 않도록 제한.

예

```text
Backend Pod 3개
minAvailable: 2

→ Drain 중에도 최소 2개 유지
```

---

# Monitoring

## monitoring.serviceMonitor

```yaml
monitoring:
  serviceMonitor:
    enabled: false
    interval: 30s
    path: /metrics
    port: http
    additionalLabels: {}
```

Prometheus Operator가 Application Metric을 수집하도록 연결하는 Resource.

```text
Application /metrics
        ↓
Service
        ↓
ServiceMonitor
        ↓
Prometheus
        ↓
Grafana
```

예

```text
Backend → /actuator/prometheus
AI      → /metrics
```

### `additionalLabels`

Prometheus가 특정 ServiceMonitor만 선택하도록 Label을 추가할 때 사용.

---

# Network

## networkPolicy

```yaml
networkPolicy:
  enabled: false
  policyTypes:
    - Ingress
  ingress: []
  egress: []
```

Pod 간 Network 접근 범위를 제한.

### `policyTypes`

어느 방향의 Traffic을 제어할지 지정.

```text
Ingress
→ 외부 / 다른 Pod에서 해당 Pod로 들어오는 Traffic

Egress
→ 해당 Pod에서 외부 / 다른 Pod로 나가는 Traffic
```

예

```text
Backend → AI API   O
Frontend → AI API  X
Internet → AI API  X
```

세부 정책 합의 후 활성화.

---

# 먼저 이해할 항목

처음에는 아래 순서로 확인.

```text
1. app
2. replicaCount
3. image
4. container
5. service
6. resources
```

그다음 Kubernetes 운영 개념 확인.

```text
7. imagePullSecrets
8. serviceAccount
9. podLabels / podAnnotations
10. SecurityContext
11. Probe
12. nodeSelector / tolerations / affinity
13. Ingress
14. HPA / KEDA
15. PDB
16. ServiceMonitor
17. NetworkPolicy
```

핵심 흐름

```text
values.yaml
   ↓
Helm Template
   ↓
Deployment / Service / Ingress 등 Manifest 생성
   ↓
Kubernetes
```
