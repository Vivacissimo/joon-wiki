---
title: HandsOn_Project
description: 
published: true
date: 2026-07-11T05:10:39.370Z
tags: 
editor: markdown
dateCreated: 2026-06-30T08:42:01.483Z
---


# ⚒️ 사용 기술 및 개발 환경

## Development

| 기술 | 사용 목적 |
|---|---|
| ![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB) | 회원가입·로그인, 호텔 검색, 호텔 상세 조회, 특가 이벤트 및 예약 화면 구현 |
| ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white) | 회원·호텔 검색·특가 이벤트·예약 기능을 담당하는 마이크로서비스 개발 |
| ![Java 21](https://img.shields.io/badge/Java_21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white) | 백엔드 애플리케이션 개발 |
| ![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=springsecurity&logoColor=white) | 로그인, 비밀번호 암호화 및 사용자 인증 처리 |
| ![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white) | 로그인 사용자의 인증 정보와 서비스 접근 권한 전달 |
| ![OpenAPI](https://img.shields.io/badge/OpenAPI-6BA539?style=for-the-badge&logo=openapiinitiative&logoColor=white) <br> ![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=black) | 서비스별 API 명세 작성 및 API 동작 테스트 |
| ![Gradle](https://img.shields.io/badge/Gradle-02303A?style=for-the-badge&logo=gradle&logoColor=white) | Java 프로젝트 빌드 및 의존성 관리 |
| ![Git](https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white) <br> ![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white) | 소스 코드 형상 관리 및 팀 협업 |

---

## Database

| 기술 | 사용 목적 |
|---|---|
| ![PostgreSQL](https://img.shields.io/badge/postgresql-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white) | 회원, 호텔, 객실, 특가 이벤트 및 예약 정보 저장 |
| ![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white) | 호텔 검색 결과 캐싱, 특가 객실 재고, 중복 예약 방지 및 가상 대기열 정보 관리 |
| ![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-000000?style=for-the-badge&logo=apachekafka&logoColor=white) | 예약·결제 결과, 예약 상태 변경 및 객실 재고 복구 이벤트 비동기 처리 |

> Redis 기반 가상 대기열과 Kafka 기반 비동기 처리는 기본 기능 완료 이후 확장 기능으로 적용한다.

---

## Server

| 기술 | 사용 목적 |
|---|---|
| ![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white) | 호텔 예약 서비스와 Observability 시스템을 운영할 클라우드 환경 제공 |
| ![Terraform](https://img.shields.io/badge/Terraform-844FBA?style=for-the-badge&logo=terraform&logoColor=white) | VPC, EKS 및 관련 AWS 인프라 생성·삭제 자동화 |
| ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) | 프론트엔드와 마이크로서비스별 컨테이너 이미지 생성 |
| ![Amazon EKS](https://img.shields.io/badge/Amazon_EKS-FF9900?style=for-the-badge&logo=amazoneks&logoColor=white) | Kubernetes 기반 호텔 예약 서비스 운영 환경 구성 |
| ![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white) | 마이크로서비스 배포, 확장, 장애 복구 및 운영 |
| ![Helm](https://img.shields.io/badge/helm-%230F1689.svg?style=for-the-badge&logo=helm&logoColor=white) | 서비스와 Observability 환경의 Kubernetes 배포 설정 관리 |
| ![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white) | 외부 요청을 프론트엔드와 백엔드 서비스로 라우팅 |
| ![Kubernetes HPA](https://img.shields.io/badge/Kubernetes_HPA-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white) | 호텔 특가 이벤트 트래픽 증가 시 서비스 Pod 자동 확장 |
| ![ConfigMap](https://img.shields.io/badge/ConfigMap-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white) <br> ![Secret](https://img.shields.io/badge/Secret-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white) | 일반 설정값과 데이터베이스 비밀번호 등의 민감 정보 분리 관리 |
| ![Istio](https://img.shields.io/badge/Istio-466BB0?style=for-the-badge&logo=istio&logoColor=white) | 서비스 간 트래픽 관리, 통신 관찰 및 장애 상황 분석 |
| ![Kiali](https://img.shields.io/badge/Kiali-2A3F54?style=for-the-badge&logo=kiali&logoColor=white) | 호텔 예약 서비스 간 호출 관계와 트래픽 흐름 시각화 |

---

## Test

| 기술 | 사용 목적 |
|---|---|
| ![JUnit 5](https://img.shields.io/badge/JUnit5-F5F5F5?style=for-the-badge&logo=junit5&logoColor=DC524A) | 회원, 호텔 검색, 특가 이벤트, 예약 및 객실 재고 로직 단위 테스트 |
| ![JaCoCo](https://img.shields.io/badge/JaCoCo-C71A36?style=for-the-badge&logo=java&logoColor=white) | Java 테스트 코드의 Line·Branch·Method 커버리지 측정 |
| ![Testcontainers](https://img.shields.io/badge/Testcontainers-2496ED?style=for-the-badge&logo=docker&logoColor=white) | PostgreSQL·Redis·Kafka를 이용한 실제 환경 기반 통합 테스트 |
| ![K6](https://img.shields.io/badge/K6-7D64FF?style=for-the-badge&logo=k6&logoColor=white) | 호텔 특가 이벤트 시작 시 다수의 동시 사용자와 급격한 예약 트래픽 발생 |
| ![Postman](https://img.shields.io/badge/Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white) | 서비스별 API 요청 및 응답 테스트 |
| ![Chaos Mesh](https://img.shields.io/badge/Chaos_Mesh-0066CC?style=for-the-badge&logo=kubernetes&logoColor=white) | Pod 종료, CPU 부하, 네트워크 지연 등의 장애 주입 및 복구 검증 |

---

## Monitoring

| 기술 | 사용 목적 |
|---|---|
| ![Spring Boot Actuator](https://img.shields.io/badge/Spring_Boot_Actuator-6DB33F?style=for-the-badge&logo=springboot&logoColor=white) | 서비스 상태, JVM 및 애플리케이션 메트릭 제공 |
| ![Micrometer](https://img.shields.io/badge/Micrometer-1F4E79?style=for-the-badge&logo=springboot&logoColor=white) | 호텔 검색·특가 조회·예약 요청 등의 애플리케이션 메트릭 생성 |
| ![OpenTelemetry SDK](https://img.shields.io/badge/OpenTelemetry_SDK-FFFFFF?style=for-the-badge&logo=opentelemetry&logoColor=black) <br> ![Java Agent](https://img.shields.io/badge/Java_Agent-FFFFFF?style=for-the-badge&logo=opentelemetry&logoColor=black) | 애플리케이션의 메트릭·로그·Trace 생성 및 서비스 간 예약 요청 추적 |
| ![OpenTelemetry Collector](https://img.shields.io/badge/OpenTelemetry_Collector-FFFFFF?style=for-the-badge&logo=opentelemetry&logoColor=black) | 각 서비스에서 생성된 관찰성 데이터를 통합 수집하고 저장소로 전달 |
| ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white) | 호텔 예약 서비스 및 Kubernetes 인프라 메트릭 수집·저장 |
| ![Loki](https://img.shields.io/badge/Loki-F46800?style=for-the-badge&logo=grafana&logoColor=white) | 회원·검색·특가 이벤트·예약 서비스에서 생성된 구조화 로그 저장 |
| ![Tempo](https://img.shields.io/badge/Tempo-F46800?style=for-the-badge&logo=grafana&logoColor=white) | 호텔 검색부터 예약 완료까지 서비스 간 분산 Trace와 Span 정보 저장 |
| ![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white) | 메트릭·로그·Trace 통합 조회 및 4 Golden Signals 대시보드 구성 |
| ![Alertmanager](https://img.shields.io/badge/Alertmanager-E6522C?style=for-the-badge&logo=prometheus&logoColor=white) | 오류율, 응답시간, 가용성, 예약 실패율 및 자원 사용량 알림 관리 |
| ![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white) | 서비스 장애, SLO 위반 및 시스템 이상 알림 전달 |

---

## CI/CD

| 기술 | 사용 목적 |
|---|---|
| ![GitHub Actions](https://img.shields.io/badge/github_actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white) | 변경된 서비스의 소스 코드 테스트, 이미지 빌드, 보안 검사 및 ECR 업로드 자동화 |
| ![Amazon ECR](https://img.shields.io/badge/Amazon_ECR-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white) | 호텔 마이크로서비스별 컨테이너 이미지 저장 및 버전 관리 |
| ![Argo CD](https://img.shields.io/badge/Argo_CD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white) | Git 저장소의 Helm 배포 설정을 Amazon EKS 환경에 자동 반영 |
| ![Helm](https://img.shields.io/badge/Helm_Chart-0F1689?style=for-the-badge&logo=helm&logoColor=white) | 서비스별 Kubernetes 리소스와 개발·운영 환경 설정 관리 |
| ![Trivy](https://img.shields.io/badge/Trivy-1904DA?style=for-the-badge&logo=aquasecurity&logoColor=white) | 컨테이너 이미지와 프로젝트 의존성의 보안 취약점 검사 |

---

## 서비스별 적용 기술

| 서비스 | 주요 역할 | 적용 기술 |
|---|---|---|
| **Frontend** | 회원가입·로그인, 호텔 검색, 호텔 상세 조회, 특가 이벤트 카운트다운 및 예약 결과 화면 | ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) ![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white) |
| **User Service** | 회원가입, 로그인, 사용자 정보 및 사용자별 예약 내역 관리 | ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![JWT](https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white) |
| **Search Service** | 지역·날짜 기반 호텔 검색, 호텔 상세 조회, 객실 및 가격 정보 제공 | ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DD0031?style=flat-square&logo=redis&logoColor=white) |
| **Event Service** | 호텔 한정 특가 이벤트, 이벤트 시간, 특가 가격 및 잔여 객실 수량 관리 | ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DD0031?style=flat-square&logo=redis&logoColor=white) |
| **Booking Service** | 예약 가능 여부 확인, 객실 재고 차감, 예약 생성 및 예약 상태 관리 | ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![Kafka](https://img.shields.io/badge/Kafka-000000?style=flat-square&logo=apachekafka&logoColor=white) |
| **Queue Service** | 특가 예약 대기열 등록, 대기 순번 조회, 순차 입장 및 예약 권한 발급 | ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DD0031?style=flat-square&logo=redis&logoColor=white) |
| **Payment Service** | 테스트 결제 성공·실패·지연 처리 및 결제 결과 전달 | ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![Kafka](https://img.shields.io/badge/Kafka-000000?style=flat-square&logo=apachekafka&logoColor=white) |
| **Observability System** | 메트릭·로그·Trace 수집, 통합 분석, 대시보드 구성 및 장애 알림 | ![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-FFFFFF?style=flat-square&logo=opentelemetry&logoColor=black) ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white) ![Loki](https://img.shields.io/badge/Loki-F46800?style=flat-square&logo=grafana&logoColor=white) ![Tempo](https://img.shields.io/badge/Tempo-F46800?style=flat-square&logo=grafana&logoColor=white) ![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white) |

> 현재는 `User Service`, `Search Service`, `Event Service`를 중심으로 구성하고, `Booking Service`, `Queue Service`, `Payment Service`는 프로젝트 진행 상황에 따라 단계적으로 확장한다.
