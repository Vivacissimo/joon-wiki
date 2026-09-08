---
title: Helm Chart.yaml
description: MoonCheap Helm Chart.yaml 항목 정리
published: true
date: 2026-09-08T07:58:00.000Z
tags: mooncheap,helm,chart
editor: markdown
dateCreated: 2026-09-08T07:58:00.000Z
---

# Chart.yaml

## 역할

Helm Chart 자체의 기본 정보 정의.

실제 Pod 개수, Image, Port 같은 배포 설정은 `values.yaml`과 `templates/`에서 관리.

---

## 현재 구성

```yaml
apiVersion: v2
name: moongcheap-service
description: Reusable Helm chart for MoongCheap application workloads on EKS
type: application
version: 0.1.0
appVersion: "0.1.0"
```

---

## 항목

### apiVersion

```yaml
apiVersion: v2
```

- Helm Chart 포맷 버전
- Helm 3 기준

### name

```yaml
name: moongcheap-service
```

- Chart 이름
- 공통 Application Chart 이름

### description

```yaml
description: Reusable Helm chart for MoongCheap application workloads on EKS
```

- Chart 용도 설명
- 배포 동작에는 직접 영향 없음

### type

```yaml
type: application
```

- 실제 Application 배포용 Chart

### version

```yaml
version: 0.1.0
```

- Helm Chart 자체 버전
- Template이나 Chart 구조 변경 시 증가

### appVersion

```yaml
appVersion: "0.1.0"
```

- Chart가 배포하는 Application 버전 정보
- 현재 프로젝트에서는 실제 배포 추적을 Image Tag의 Git Commit SHA로 관리

---

## version / appVersion 차이

```text
version
→ Helm Chart 버전

appVersion
→ Application 버전 정보
```

MoonCheap에서는 실제 배포 Image 기준

```yaml
image:
  tag: a82f91c
```

Git Commit SHA를 주요 추적값으로 사용.
