---
title: HandsOn_Project
description: 
published: true
date: 2026-06-30T08:53:48.243Z
tags: 
editor: markdown
dateCreated: 2026-06-30T08:42:01.483Z
---


# ⚒️ 사용 기술 및 개발 환경

## Development







![Gradle](https://img.shields.io/badge/Gradle-02303A.svg?style=for-the-badge&logo=gradle&logoColor=white)

| 기술 | 사용 목적 |
|---|---|
| ![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB) | 상품 조회, 회원가입, 장바구니, 대기 순번, 주문·결제 화면 구현 |
| ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white) | Commerce·Queue·Payment 마이크로서비스 개발 |
| ![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white) | 백엔드 애플리케이션 개발 |
| ![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=springsecurity&logoColor=white) | 로그인 및 사용자 인증 처리 |
| ![JWT](https://img.shields.io/badge/JWT-black?style=for-the-badge&logo=JSON%20web%20tokens) | 로그인 사용자 인증 정보 전달 |
| **OpenAPI** / ![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=black) | 서비스별 API 명세 및 테스트 |
| ![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white) | 소스 코드 형상 관리 및 팀 협업 |

---

## Database

| 기술 | 사용 목적 |
|---|---|
| **PostgreSQL** | 회원, 상품, 장바구니, 주문, 결제 정보 저장 |
| **Redis** | 가상 대기열, 대기 순번, 입장 권한, 한정상품 재고 관리 |
| **Apache Kafka** | 주문·결제·재고 복구 이벤트 비동기 처리 |

---

## Server

| 기술 | 사용 목적 |
|---|---|
| **Docker** | 서비스별 컨테이너 이미지 생성 |
| **Kubernetes** | 마이크로서비스 배포 및 운영 |
| **Helm** | 서비스 및 Observability 환경 배포 설정 관리 |
| **Nginx Ingress Controller** | 외부 요청을 프론트엔드와 백엔드 서비스로 라우팅 |
| **Kubernetes HPA** | 트래픽 증가 시 서비스 인스턴스 자동 확장 |
| **Kubernetes ConfigMap / Secret** | 일반 설정값과 민감 정보 분리 관리 |

---

## Test

| 기술 | 사용 목적 |
|---|---|
| **JUnit 5** | 회원, 주문, 결제, 재고 및 대기열 로직 단위 테스트 |
| **JaCoCo** | Java 테스트 코드 커버리지 측정 |
| **Testcontainers** | PostgreSQL·Redis·Kafka 기반 통합 테스트 |
| **Locust** | 한정판매 시작 시 동시 사용자 및 대규모 트래픽 발생 |
| **Chaos Mesh** | 파드 종료, CPU 부하, 네트워크 지연 등 장애 주입 |

---

## Monitoring

| 기술 | 사용 목적 |
|---|---|
| **OpenTelemetry SDK / Java Agent** | 애플리케이션 메트릭·로그·Trace 생성 |
| **OpenTelemetry Collector** | 각 서비스의 관찰성 데이터를 통합 수집 및 전달 |
| **Prometheus** | 서비스 및 인프라 메트릭 저장 |
| **Loki** | Commerce·Queue·Payment Service의 구조화 로그 저장 |
| **Tempo** | 서비스 간 분산 Trace 저장 |
| **Grafana** | 메트릭·로그·Trace 통합 조회 및 대시보드 구성 |
| **Alertmanager** | 오류율, 응답시간, 가용성, 자원 사용량 알림 관리 |
| **Slack Webhook** | 장애 및 SLO 관련 알림 전달 |

---

## CI/CD

| 기술 | 사용 목적 |
|---|---|
| **GitHub Actions** | 빌드, 테스트, 이미지 생성 및 레지스트리 업로드 자동화 |
| **Docker Registry** | 서비스별 컨테이너 이미지 저장 |
| **ArgoCD** | Git 저장소의 배포 설정을 Kubernetes에 자동 반영 |
| **Helm Chart** | 서비스별 Kubernetes 리소스와 환경 설정 관리 |
| **Trivy** | 컨테이너 이미지 취약점 검사 |

---

## 서비스별 적용 기술

| 서비스 | 주요 역할 | 관련 기술 |
|---|---|---|
| **Frontend** | 상품, 장바구니, 대기 순번, 주문·결제 화면 | React |
| **Commerce Service** | 회원, 상품, 장바구니, 주문, 재고 | Spring Boot, PostgreSQL |
| **Queue Service** | 대기열 등록, 순번 조회, 구매 권한 발급 | Spring Boot, Redis |
| **Payment Service** | 테스트 결제, 실패·지연 처리, 결제 결과 전달 | Spring Boot, PostgreSQL, Kafka |
| **Event Consumer** | 결제 결과 반영, 주문 상태 변경, 재고 복구 | Kafka, Spring Boot |
| **Observability System** | 메트릭·로그·Trace 수집 및 장애 분석 | OpenTelemetry, Prometheus, Loki, Tempo, Grafana |


## ⚒️ 사용 기술 및 개발 환경



---

### Database

![PostgreSQL](https://img.shields.io/badge/postgresql-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-000?style=for-the-badge&logo=apachekafka)

---

### Server

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/helm-%230F1689.svg?style=for-the-badge&logo=helm&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)

---

### Test

![JUnit5](https://img.shields.io/badge/JUnit5-f5f5f5?style=for-the-badge&logo=junit5&logoColor=dc524a)
![JaCoCo](https://img.shields.io/badge/JaCoCo-C71A36?style=for-the-badge)
![Testcontainers](https://img.shields.io/badge/Testcontainers-2496ED?style=for-the-badge&logo=testcontainers&logoColor=white)
![Locust](https://img.shields.io/badge/Locust-1B4D3E?style=for-the-badge&logo=locust&logoColor=white)
![Postman](https://img.shields.io/badge/Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white)
![Chaos Mesh](https://img.shields.io/badge/Chaos_Mesh-0066CC?style=for-the-badge)

---

### Monitoring

![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-FFFFFF?style=for-the-badge&logo=opentelemetry&logoColor=black)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Tempo](https://img.shields.io/badge/Tempo-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Alertmanager](https://img.shields.io/badge/Alertmanager-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)

---

### CI/CD

![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![Argo CD](https://img.shields.io/badge/Argo_CD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy-1904DA?style=for-the-badge&logo=aquasecurity&logoColor=white)
![GitHub Container Registry](https://img.shields.io/badge/GitHub_Container_Registry-181717?style=for-the-badge&logo=github&logoColor=white)




