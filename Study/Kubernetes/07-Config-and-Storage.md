---
title: 07. Config and Storage
description: ConfigMap Secret Volume Storage
published: true
tags: kubernetes
editor: markdown
---

# Config와 Storage

Application 설정값과 데이터를 Container Image에서 분리해서 관리.

## ConfigMap

일반 설정값 저장.

예시:

- API URL
- Log Level
- Feature 설정

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: info
```

Pod에서 환경변수 또는 파일 형태로 사용 가능.

## Secret

민감한 설정값 관리.

예시:

- Password
- Token
- API Key
- 인증서

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: example
```

Secret은 기본적으로 안전한 비밀 저장소 자체를 의미하지 않음.

RBAC, Encryption at Rest, 외부 Secret Manager 등을 같이 고려.

## Volume

Pod 내부 데이터를 저장하거나 Container 간 데이터를 공유하기 위한 저장 공간.

Pod 삭제 시 함께 사라지는 Volume도 있고 외부 Storage와 연결되는 Volume도 있음.

## PV

PersistentVolume.

Cluster에서 사용할 영구 Storage 자원.

## PVC

PersistentVolumeClaim.

Pod가 필요한 Storage를 요청하는 리소스.

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

## StorageClass

PVC 요청에 맞춰 PV를 동적으로 생성하기 위한 Storage 정책.

Cloud 환경 예시:

```text
PVC
 ↓
StorageClass
 ↓
AWS EBS CSI
 ↓
EBS Volume
```

## 확인 명령

```bash
kubectl get configmap
kubectl get secret
kubectl get pv
kubectl get pvc
kubectl get storageclass
```

## 다음

`08-Resource-and-Scheduling`에서 CPU/Memory와 Pod 배치 기준 정리.
