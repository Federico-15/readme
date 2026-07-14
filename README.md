<div align="center">

# 콩콩팥팥

### 농민을 위한 대안 신용평가 기반 BNPL 플랫폼

농업 데이터를 바탕으로 농민에게 맞춤형 구매 한도를 제공하고,<br />
시계열 예측 기반 오토스케일링과 통합 관측성을 갖춘 하이브리드 클라우드 시스템입니다.

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white)
![Argo CD](https://img.shields.io/badge/Argo_CD-EF7B4D?style=flat-square&logo=argo&logoColor=white)

<br />

<img src="./docs/images/cover.png" alt="콩콩팥팥 프로젝트 소개" width="900" />

</div>

---

## 📑 목차

1. [프로젝트 개요](#-프로젝트-개요)
2. [대안 신용평가 모델](#-대안-신용평가-모델)
3. [주요 서비스 기능](#-주요-서비스-기능)
4. [시스템 아키텍처](#️-시스템-아키텍처)
5. [인프라 및 기술적 특징](#️-인프라-및-기술적-특징)
6. [성능 개선](#-성능-개선)
7. [트러블슈팅](#-트러블슈팅)
8. [테스트 및 품질](#-테스트-및-품질)
9. [팀원 소개](#-팀원-소개)
10. [리포지토리](#-리포지토리)
11. [향후 과제](#-향후-과제)
12. [기술 스택](#️-기술-스택)

---

## 📌 프로젝트 개요

### 기획 배경 - 문제 상황

#### 1. 고정 급여가 없어 기존 신용평가만으로 상환 능력을 증명하기 어려움

농민은 일반 근로자와 달리 매월 일정한 급여를 받지 않는 경우가 많아, 소득과 금융 거래 이력을 중심으로 평가하는 기존 신용평가만으로는 실제 상환 능력을 충분히 인정받기 어렵습니다.

#### 2. 농업의 계절성과 영농 활동이 신용평가에 충분히 반영되지 않음

농업 소득은 작물의 재배·수확 시기에 따라 크게 달라지며, 경작 면적이나 작물재해보험 가입 여부와 같은 실제 영농 정보도 기존 금융 데이터에 제대로 기록되지 않아 농민의 신용도를 정확히 평가하기 어렵습니다.

#### 3. 농자재 구매 시점과 수확 소득 발생 시점의 차이로 자금 부담이 발생

농민은 종자, 비료, 농약 등 영농에 필요한 농자재를 수확 전에 구매해야 하지만 소득은 수확 이후에 발생합니다. 이로 인해 영농 초기에 자금 부담이 집중되고, 필요한 시점에 적절한 금융 지원을 받기 어렵습니다.

### 프로젝트 목표

- **농업 특화 대안 신용평가**: 경작 면적, 작물재해보험 가입 여부, 주 재배 품목과 상환 행동을 반영해 농민의 신용도를 평가합니다.
- **농민 전용 BNPL 서비스**: 농자재 구매부터 월별 이자 납부, 원금 상환까지 하나의 흐름으로 제공합니다.
- **하이브리드 클라우드**: 금융 원장과 민감 데이터는 온프레미스에, 확장성과 접근성이 필요한 서비스는 AWS에 배치합니다.
- **예측형 운영 자동화**: 과거 트래픽을 학습한 GRU 모델과 KEDA를 결합해 트래픽 증가 전에 리소스를 확장합니다.
- **AIOps와 통합 관측성**: 로그, 메트릭, 트레이스와 내부 토폴로지를 활용해 장애 원인과 조치 방향을 자동 분석합니다.

> 우리 FISA 클라우드 엔지니어링 5기 최종 프로젝트 - 데브식스

---

## 🧮 대안 신용평가 모델

콩콩팥팥은 신규 가입자의 정적 농업 정보와 서비스 이용 이후의 행동 데이터를 단계적으로 결합합니다.

| 점수 | 평가 시점 | 주요 평가 데이터 | 역할 |
| --- | --- | --- | --- |
| **ASS** | 신규 가입 | 농업경영체 등록 기반 경작 면적, 농작물 재해보험 가입 여부, 주 재배 품목 | 초기 신용 점수 산정 |
| **BSS** | 한도 승인 후 | 이자·원금 상환 이력, 연체 및 연체 해소 이력, 한도 사용률 | 월별 행동 점수 산정 |
| **CSS** | 최종 평가 | 신규 신청자는 ASS, 기존 이용자는 ASS와 BSS를 함께 반영 | 최종 신용 점수와 BNPL 한도 결정 |

### 결제 워크플로

```text
최초 거래
ASS 및 CSS 산출
  -> BNPL 거래 및 월별 이자 납부
  -> 월별 이자 납부와 연체 현황을 반영해 BSS 산출
  -> 원금 상환 및 거래 종료

재거래
ASS와 기존 BSS를 합산해 CSS 갱신
  -> BNPL 거래 및 월별 이자 납부
  -> 행동 데이터를 반영해 BSS 갱신
  -> 원금 상환 및 거래 종료
```

---

## ✨ 주요 서비스 기능

| 기능 | 설명 |
| --- | --- |
| **사용자·인증** | 회원가입, 로그인, 인증과 결제 PIN 검증을 제공합니다. |
| **대안 신용평가** | 농업 정보와 상환 행동을 바탕으로 신용 점수, 산출 근거와 최종 결과를 제공합니다. |
| **농부 프로필** | 기본 정보, 작물 종류, 경작 면적과 보험 가입 정보를 관리합니다. |
| **한도 관리** | 신용평가 결과에 따라 BNPL 한도를 부여하고 사용·잔여 한도를 관리합니다. |
| **상품 주문·조회** | 농자재 상품을 조회하고 BNPL 주문과 주문 상태를 관리합니다. |
| **원장 관리** | 대출·이자·연체 원장을 생성하고 상환 내역을 기록합니다. |
| **상환 및 연체** | 만기 기한에 따라 원금과 이자를 상환하고 연체 발생·해소 이력을 반영합니다. |
| **관리자 대시보드** | 농부, 한도, 주문, 상환, 연체, 원장과 감사 로그를 통합 조회합니다. |
| **AIOps** | 운영 데이터로 이상을 탐지하고 LLM 기반 장애 원인 분석과 조치 가이드를 제공합니다. |

### 🎬 서비스 시연 플로우

각 미리보기를 클릭하면 전체 시연 영상을 확인할 수 있습니다.

### 🎬 서비스 시연 플로우

각 GIF 미리보기를 클릭하면 전체 시연 영상을 확인할 수 있습니다.

#### 👨‍🌾 사용자 측면

| 🧾 한도 신청 | 🛒 농자재 구매 | 💬 유저 챗봇 |
| :---: | :---: | :---: |
| <a href="https://github.com/user-attachments/assets/27c1b67d-1511-4968-8b9c-f509bc1c9831"><img src="https://github.com/user-attachments/assets/ad4e8121-5509-477e-9f64-9aeafa4d64c6" alt="사용자 한도 신청 시연" width="260" /></a> | <a href="https://github.com/user-attachments/assets/4456f7fb-66cf-4036-871f-073224e98be2"><img src="https://github.com/user-attachments/assets/d7c5cb5e-c34e-4313-90db-cbf1a624aa82" alt="사용자 농자재 구매 시연" width="260" /></a> | <a href="https://github.com/user-attachments/assets/707d4d3f-d6cc-45f0-816d-070a0c706d6b"><img src="https://github.com/user-attachments/assets/079e8a28-1fe2-4eb2-8059-71e483ce8c4c" alt="사용자 챗봇 시연" width="260" /></a> |

#### 👩‍💼 관리자 측면

| ✅ 한도 승인 | 🚚 배송 관리 | ⚠️ 연체 현황 | 🤖 관리자 챗봇 |
| :---: | :---: | :---: | :---: |
| <a href="https://github.com/user-attachments/assets/b009d282-8943-45aa-ae74-42b32a153303"><img src="https://github.com/user-attachments/assets/615850e9-94f2-4a62-8000-76b9cce07bff" alt="관리자 한도 승인 시연" width="220" /></a> | <a href="https://github.com/user-attachments/assets/8570502d-e58a-44f3-b3d5-bee9b9c8499c"><img src="https://github.com/user-attachments/assets/c05d06ff-d89d-4864-b9ff-420eb24cf68a" alt="관리자 배송 관리 시연" width="220" /></a> | <a href="https://github.com/user-attachments/assets/bce8c3a2-f763-49ea-bc4e-a15e3bb30557"><img src="https://github.com/user-attachments/assets/b5bf9391-f03c-42af-9789-8871551f22b7" alt="관리자 연체 현황 시연" width="220" /></a> | <a href="https://github.com/user-attachments/assets/fa1fb090-5d7e-48d5-b49d-d17bdfd32396"><img src="https://github.com/user-attachments/assets/4289822a-77ac-4f52-9667-9fa79df791f5" alt="관리자 챗봇 시연" width="220" /></a> |

---

## 🏗️ 시스템 아키텍처

AWS, 온프레미스 데이터센터와 VESSL AI를 연결한 하이브리드 아키텍처입니다. MSA 서비스는 역할과 데이터 민감도에 따라 AWS EKS와 온프레미스 Kubernetes에 분산하고, Site-to-Site VPN을 통해 사설 통신합니다.

<div align="center">
  <img src="./docs/images/system-architecture.png" alt="콩콩팥팥 하이브리드 시스템 아키텍처" width="950" />
</div>

### 영역별 구성

| 영역 | 구성 | 역할 |
| --- | --- | --- |
| **AWS Cloud** | Route 53, CloudFront, S3, ALB, EKS, ECR, RDS, DMS, SQS, CloudWatch | 정적 웹 호스팅, 외부 진입점, 확장형 서비스, 비동기 메시지, DR과 클라우드 모니터링 |
| **On-premises** | pfSense, HAProxy, Kubernetes, PostgreSQL, Patroni, etcd, TrueNAS | 금융 원장·거래 데이터, 내부 서비스, DB 고가용성과 영구 스토리지 |
| **VESSL AI** | H100 GPU, vLLM, Qwen3 | 사내 LLM 추론과 AIOps 분석 |
| **Delivery** | GitLab, Jenkins, Harbor, Trivy, Argo CD | 빌드·검증·이미지 관리와 GitOps 배포 |
| **Observability** | Prometheus, Grafana, Loki, Tempo, OpenTelemetry, Fluent Bit, Hubble | 로그, 메트릭, 트레이스와 네트워크 흐름 통합 관측 |

### 주요 설계 결정

1. **서비스와 데이터 분리**: 외부 접근과 확장이 필요한 상품·프런트 서비스는 AWS에, 금융 원장과 민감 데이터는 온프레미스에 배치했습니다.
2. **네트워크 고가용성**: pfSense를 이중화하고 허용된 IP와 포트만 통과시키며, 내부 VLAN은 HAProxy를 통해 라우팅합니다.
3. **데이터 고가용성**: 온프레미스 PostgreSQL은 Patroni와 etcd 기반 Primary-Replica 구조로 구성하고, 읽기·쓰기를 분리하는 CQRS를 적용했습니다.
4. **클라우드 DR**: RDS를 이중화하고 DMS Full Load와 CDC로 재해 복구 경로를 구성했습니다.
5. **비동기 서비스 연계**: 인증, 상품, 결제 서비스 간 이벤트를 AWS SQS로 전달해 결합도를 낮췄습니다.

---

## ☁️ 인프라 및 기술적 특징

### 1️⃣ GRU + KEDA 기반 Predictive Autoscaling

기존 HPA는 CPU·메모리 임계치를 넘은 뒤에야 확장을 시작해 트래픽 급증 시 500 오류와 긴 지연이 발생했습니다. 과거 트래픽으로 학습한 GRU 모델이 향후 RPS를 예측하고, 예측값을 KEDA 외부 메트릭으로 전달해 필요한 Pod를 미리 확보하도록 개선했습니다.

| 지표 | HPA | GRU + KEDA | 개선 |
| --- | ---: | ---: | ---: |
| 최고 RPS | 112 req/s | 91.4 req/s | 안정성 중심으로 처리량 조정 |
| P95 최고 지연 | 9.70s | 389ms | **96.0% 감소** |
| P99 최고 지연 | 9.94s | 1.55s | **84.4% 감소** |
| 500 오류 발생 구간 | 100% | 0% | **완전 제거** |

#### 🎬 HPA vs GRU + KEDA 시연 영상

##### HPA — 반응형 오토스케일링

https://github.com/user-attachments/assets/72c94b8f-1ee3-4fe1-a605-4859b15d4d99

##### GRU + KEDA — 예측형 오토스케일링

https://github.com/user-attachments/assets/0484ced5-83cc-4cdc-9110-d9cc4172cc51

### 2️⃣ MCP 기반 AIOps

- 서버 CPU, 메모리, 디스크와 서비스 트래픽을 수집합니다.
- 시계열 예측 RPS와 실제 트래픽의 차이를 감지합니다.
- 예측값 초과 시 로그, 메트릭, 트레이스와 내부 토폴로지를 함께 수집합니다.
- VESSL AI의 Qwen LLM이 MCP 도구를 통해 필요한 운영 데이터만 조회합니다.
- 서비스, 네트워크, DB 경로를 시각화하고 RCA 원인 후보와 조치 방향을 Slack으로 전달합니다.
- 민감 정보는 마스킹하고, LLM과 인프라 사이에는 실행 범위를 제한하는 MCP 계층을 둡니다.

### 3️⃣ 통합 관측성과 알림

- **EKS**: CloudWatch로 노드·Pod CPU 및 메모리, 네트워크 트래픽과 오류 자원을 관측합니다.
- **온프레미스 K8s**: Prometheus 기반 리소스 모니터링과 Hubble 기반 HTTP·TCP·UDP·ICMP 네트워크 흐름을 관측합니다.
- **PostgreSQL**: 호스트 자원, 쿼리·트랜잭션 처리량, Replication Lag, Connection과 오류 로그를 관측합니다.
- **pfSense**: 이중화 상태, 자원 사용률, AWS 간 트래픽과 VLAN별 허용·차단 이벤트를 수집합니다.
- **분산 트레이싱**: OpenTelemetry로 서비스 로그와 트레이스를 연결하고 Tempo·Grafana에서 BNPL 결제 흐름을 추적합니다.
- **알림**: EKS Pod·Node 이상과 PostgreSQL CPU·메모리·디스크 임계치 초과를 Alertmanager와 Slack으로 전송합니다.

#### 🎬 통합 모니터링 시연 영상

##### 1. On-prem Kubernetes

https://github.com/user-attachments/assets/857108f3-8ed4-4993-b7f4-53bdc48fb926

##### 2. pfSense

https://github.com/user-attachments/assets/0c1cf486-4297-4ad0-a8e5-e04362c968b5

##### 3. CloudWatch

https://github.com/user-attachments/assets/b41945bf-04b9-4971-9ee6-263a6d54e093

##### 4. Alertmanager

https://github.com/user-attachments/assets/bf7ef90f-8918-43c0-984d-c831cd9a5102

##### 5. Observability — Distributed Trace

https://github.com/user-attachments/assets/76d5982a-104d-4e62-a70e-8afd41ab1696

##### 6. AIOps — 장애 분석 및 대응

https://github.com/user-attachments/assets/aec04230-9acb-4441-8e24-22f434c584fc

##### 7. AIOps — 운영 분석 및 조치 가이드

https://github.com/user-attachments/assets/92df2fc6-2c2b-479b-9663-ccb6d69fe8b3

### 4️⃣ GitOps 기반 배포와 공급망 보안

```text
Developer
  -> GitLab
  -> Jenkins: Build / Test / Image Build
  -> Harbor + Trivy: Image Registry / Vulnerability Scan
  -> GitLab GitOps Repository
  -> Argo CD
  -> Kubernetes Services
```

Harbor에는 온프레미스 이미지를 저장하고 Trivy로 취약점을 검사합니다. AWS 서비스 이미지는 ECR에 저장해 EKS로 배포하며, Argo CD가 GitOps 저장소의 선언 상태와 클러스터 상태를 동기화합니다.

---

## 🚀 성능 개선

### CQRS 적용

PostgreSQL 읽기와 쓰기 책임을 분리하고 Patroni 기반 Primary-Replica 구조와 연계했습니다. k6로 100 RPS를 10분간 발생시켜 개선 전후를 비교했습니다.

| 지표 | 적용 전 | 적용 후 | 개선 |
| --- | ---: | ---: | ---: |
| 평균 응답 시간 | 50.48ms | 35.60ms | **29.5% 감소** |
| P95 응답 시간 | 68.05ms | 51.28ms | **24.6% 감소** |
| 최대 응답 시간 | 4.01s | 1.38s | **65.6% 감소** |
| Dropped Iterations | 304건 | 21건 | **93.1% 감소** |

### 컨테이너 이미지 최적화

경량 런타임 베이스 이미지, Multi-stage Build와 Spring Boot Layered JAR를 적용했습니다. 빌드 환경과 실행 환경을 분리하고, 의존성 레이어와 애플리케이션 레이어를 나눠 Docker 캐시 재사용률을 높였습니다.

#### Harbor 취약점 분석 및 제거

Harbor에 저장된 `service-payment` 이미지를 Trivy로 스캔해 애플리케이션 의존성과 런타임 패키지의 취약점을 단계적으로 제거했습니다. 아래 화면은 최적화 전, 1차 최적화와 최종 이미지를 한눈에 보여줍니다.

<div align="center">
  <img src="./docs/images/harbor-service-payment.png" alt="Harbor service-payment 취약점 제거 결과" width="700" />
</div>

| 단계 | 이미지 태그 | 이미지 크기 | 크기 변화 | 취약점 | 제거율 |
| --- | --- | ---: | ---: | ---: | ---: |
| **최적화 전** | `18a29dc` | 138.36MiB | 기준 | 122개 | 기준 |
| **1차 최적화** | `aa587b4` | 134.94MiB | 3.42MiB 감소 (2.47%) | 47개 | **61.48% 제거** |
| **최종 최적화** | `9e51b49` | 119.20MiB | 19.16MiB 감소 (13.85%) | 0개 | **100% 제거** |

##### 분석 및 해결 과정

1. **애플리케이션 의존성 분석**: Tomcat Embed, Spring Security, Netty 등 Java 의존성에서 발견된 취약 버전을 확인하고 수정 버전으로 갱신했습니다.
2. **런타임 패키지 분석**: OpenSSL 등 기존 베이스 이미지의 OS 패키지 취약점을 분리해 확인했습니다.
3. **이미지 구조 개선**: Multi-stage Build와 Spring Boot Layered JAR로 빌드 도구를 최종 이미지에서 제외하고 캐시 재사용 범위를 개선했습니다.
4. **경량 런타임 적용**: 실행에 필요한 구성만 포함한 경량 베이스 이미지로 교체해 남은 런타임 취약점을 제거했습니다.
5. **재스캔 검증**: 최종 태그 `9e51b49`를 다시 스캔해 `취약점 없음` 상태를 확인했습니다.

<details>
<summary><strong>Trivy에서 탐지한 주요 CVE 분석 화면 보기</strong></summary>

<br />

<img src="./docs/images/harbor-cve-analysis.png" alt="Harbor Trivy 주요 CVE 분석" width="900" />

</details>

발표자료의 전체 서비스 종합 지표에서는 이미지별로 최대 128개의 취약점이 탐지됐으며, 최종 이미지에서는 모두 0개를 달성했습니다. 위 표의 122개는 첨부 화면에 표시된 `service-payment` 이미지의 개별 측정값입니다.

#### Harbor 전체 최적화 성과

| 지표 | 적용 전 | 적용 후 | 개선 |
| --- | ---: | ---: | ---: |
| Harbor 이미지 크기 | 842MB | 694MB | **17.5% 감소** |
| Harbor 최대 탐지 취약점 | 128개 | 0개 | **100% 제거** |
| 이미지 레이어 | 48개 | 32개 | **33.3% 감소** |
| 이미지 빌드 시간 | 3분 41초 | 1분 21초 | **63% 단축** |

#### ECR 이미지 최적화 전후

<table>
  <tr>
    <td align="center" width="50%">
      <strong>최적화 전</strong><br /><br />
      <img src="./docs/images/ecr-before.png" alt="ECR 이미지 최적화 전" width="360" />
    </td>
    <td align="center" width="50%">
      <strong>최적화 후</strong><br /><br />
      <img src="./docs/images/ecr-after.png" alt="ECR 이미지 최적화 후" width="360" />
    </td>
  </tr>
</table>

| 비교 항목 | 최적화 전 | 최적화 후 | 분석 |
| --- | ---: | ---: | --- |
| **이미지 크기** | 97.11MB | 49.01MB | **48.10MB 감소 (49.53% 절감)** |
| **Critical 취약점** | 1개 | 0개 | **100% 제거** |
| **High 취약점** | 3개 | 0개 | **100% 제거** |
| **Medium 취약점** | 3개 | 0개 | **100% 제거** |
| **전체 취약점** | 7개 | 0개 | **스캔 대상 취약점 완전 제거** |
| **이미지 상태** | 활성 | 활성 | 최적화 후에도 정상 배포 가능한 상태 유지 |

이미지 용량을 절반 가까이 줄여 ECR 저장 공간과 EKS 노드의 이미지 Pull 부담을 낮췄습니다. 동시에 스캔에서 발견된 Critical, High, Medium 취약점을 모두 제거해 배포 효율과 공급망 보안을 함께 개선했습니다.

### Jenkins 빌드 파이프라인 최적화

Jenkins와 Docker에서 반복 수행되던 Gradle 빌드를 제거했습니다. Jenkins가 생성한 JAR를 Docker가 패키징만 하도록 변경하고 Layered JAR 캐시를 활용했습니다.

| 구간 | 적용 전 | 적용 후 | 개선 |
| --- | ---: | ---: | ---: |
| Build | 32초 | 9초 | **71% 단축** |
| Docker Build & Push | 2분 5초 | 45초 | **64% 단축** |
| 전체 CI/CD 파이프라인 | 5분 27초 | 3분 10초 | **42% 단축** |

### LLM 입력 최적화

응답 목적 중심으로 프롬프트를 경량화하고 반복 조회 데이터에 30초 TTL 캐시를 적용했습니다.

| 지표 | 최적화 전 | 최적화 후 | 개선 |
| --- | ---: | ---: | ---: |
| 평균 응답 시간 | 3.17s | 3.07s | **3.15% 감소** |
| 요청당 평균 입력 토큰 | 1.18k | 1.06k | **10.17% 감소** |
| P95 응답 시간 | 8.67s | 8.17s | **5.77% 감소** |

---

## 🔧 트러블슈팅

<details>
<summary><strong>1️⃣ HPA의 사후 확장으로 발생한 500 오류</strong></summary>

### 문제

실제 트래픽 유입보다 늦게 Pod가 확장되면서 Scale-out이 완료될 때까지 500 오류가 발생했습니다.

### 원인

HPA는 CPU·메모리 임계치를 초과한 이후에 반응하므로 급격한 트래픽 변화에 선제적으로 대응할 수 없었습니다.

### 해결

과거 트래픽을 학습한 GRU 모델로 향후 RPS를 예측하고, KEDA가 예측 결과에 맞춰 필요한 Pod를 미리 확장하도록 구성했습니다.

### 결과

P95 최고 지연을 9.70초에서 389ms로 줄였고, 테스트 구간의 500 오류를 완전히 제거했습니다.

</details>

<details>
<summary><strong>2️⃣ 컨테이너 이미지 비대화와 보안 취약점</strong></summary>

### 문제

실행에 필요하지 않은 패키지와 취약한 베이스 이미지로 인해 이미지 용량과 ECR 비용이 증가했고, Harbor에서 128개 취약점이 발견됐습니다.

### 원인

빌드 도구와 실행 환경이 한 이미지에 포함됐으며, 실행 JAR 전체가 하나의 레이어로 복사돼 작은 코드 변경에도 큰 레이어가 다시 생성됐습니다.

### 해결

경량 런타임 베이스 이미지, Multi-stage Build와 Spring Boot Layered JAR를 적용해 빌드·실행 환경과 의존성·애플리케이션 레이어를 분리했습니다.

### 결과

Harbor 취약점을 128개에서 0개로 줄이고 이미지 빌드 시간을 63% 단축했습니다. ECR 이미지도 97.11MB에서 49.01MB로 줄였으며 Critical·High·Medium 취약점을 모두 제거했습니다.

</details>

<details>
<summary><strong>3️⃣ Jenkins와 Docker의 중복 빌드</strong></summary>

### 문제

Jenkins 빌드 이후 Docker 이미지 생성 과정에서 Gradle 빌드를 다시 수행해 파이프라인이 불필요하게 길어졌습니다.

### 해결

Jenkins가 만든 JAR를 Docker가 패키징만 하도록 변경하고, 변경 빈도가 낮은 의존성 레이어는 Harbor 캐시를 재사용하도록 구성했습니다.

### 결과

Docker Build & Push를 2분 5초에서 45초로 줄이고 전체 파이프라인을 5분 27초에서 3분 10초로 단축했습니다.

</details>

<details>
<summary><strong>4️⃣ AIOps LLM의 입력 토큰 낭비</strong></summary>

### 문제

질문과 직접 관련 없는 운영 데이터까지 프롬프트에 누적돼 토큰 사용량과 P95 응답 시간이 증가했습니다.

### 해결

프롬프트를 응답 목적 중심으로 재구성하고 반복 조회 결과에 30초 TTL 캐시를 적용했습니다.

### 결과

요청당 평균 입력 토큰을 10.17% 줄이고 P95 응답 시간을 5.77% 개선했습니다.

</details>

---

## ✅ 테스트 및 품질

Mockito와 JUnit으로 단위 테스트를 작성하고 JaCoCo로 Instruction 및 Branch Coverage를 측정했습니다. 모든 백엔드 모듈에서 두 지표 모두 90% 이상을 달성했습니다.

| 모듈 | Instruction Coverage | Branch Coverage |
| --- | ---: | ---: |
| `service-batch` | 92.73% | 90.25% |
| `service-core` | 90.16% | 90.45% |
| `service-admin` | 90.57% | 90.24% |
| `service-auth` | 95.12% | 100.00% |
| `service-catalog` | 97.32% | 93.75% |
| `service-payment` | 93.79% | 92.47% |

---

## 👥 팀원 소개

| <img src="https://github.com/Federico-15.png" width="90"/> | <img src="https://github.com/HiLeeS.png" width="90"/> | <img src="https://github.com/cuterrabbit.png" width="90"/> | <img src="https://github.com/Zaixian5.png" width="90"/> | <img src="https://github.com/ygreee0320.png" width="90"/> |
| :---: | :---: | :---: | :---: | :---: |
| **류승환** | **이승준** | **이동욱** | **사재헌** | **양규리** |
| [@Federico-15](https://github.com/Federico-15) | [@HiLeeS](https://github.com/HiLeeS) | [@cuterrabbit](https://github.com/cuterrabbit) | [@Zaixian5](https://github.com/Zaixian5) | [@ygreee0320](https://github.com/ygreee0320) |
| **PM** (AIOps & BE) | **PL** (Observability) | AIOps & Test | 모니터링 & On-prem | CI/CD & Cloud |

---

## 🔗 리포지토리

| 구분 | 설명 | 링크 |
| --- | --- | --- |
| **GitHub Organization** | 콩콩팥팥 프로젝트 전체 저장소 | [FISA-Agri-Pay](https://github.com/FISA-Agri-Pay) |
| **Infrastructure** | AWS, 온프레미스, VESSL AI 하이브리드 인프라 | [infra](https://github.com/FISA-Agri-Pay/infra) |

---

## 🔭 향후 과제

- 운영 정책을 기반으로 한 AIOps 조치 자동화
- 관리자 기능과 반복 운영 절차 자동화
- OCR과 실제 농업 데이터를 활용한 신용평가 모델 고도화
- 더 많은 금융·농업 서비스와의 연계

---

## 🛠️ 기술 스택

| 영역 | 기술 스택 |
| --- | --- |
| Frontend | ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white) ![Zustand](https://img.shields.io/badge/Zustand-764ABC?style=flat-square) |
| Backend | ![Java 21](https://img.shields.io/badge/Java%2021-007396?style=flat-square&logo=openjdk&logoColor=white) ![Spring Boot 3.5](https://img.shields.io/badge/Spring%20Boot%203.5-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![Spring Security](https://img.shields.io/badge/Spring%20Security-6DB33F?style=flat-square&logo=springsecurity&logoColor=white) ![Spring Data JPA](https://img.shields.io/badge/Spring%20Data%20JPA-6DB33F?style=flat-square&logo=spring&logoColor=white) ![Spring Batch](https://img.shields.io/badge/Spring%20Batch-6DB33F?style=flat-square&logo=spring&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) ![FastMCP](https://img.shields.io/badge/FastMCP-111827?style=flat-square) |
| Test | ![JUnit5](https://img.shields.io/badge/JUnit5-25A162?style=flat-square&logo=junit5&logoColor=white) ![Mockito](https://img.shields.io/badge/Mockito-78A641?style=flat-square) ![JaCoCo](https://img.shields.io/badge/JaCoCo-DC3545?style=flat-square) ![k6](https://img.shields.io/badge/k6-7D64FF?style=flat-square&logo=k6&logoColor=white) |
| Data / Event | ![PostgreSQL 16](https://img.shields.io/badge/PostgreSQL%2016-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white) ![AWS SQS](https://img.shields.io/badge/AWS%20SQS-FF4F8B?style=flat-square&logo=amazonsqs&logoColor=white) |
| AI / AIOps | ![Python 3.11](https://img.shields.io/badge/Python%203.11-3776AB?style=flat-square&logo=python&logoColor=white) ![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white) ![GRU](https://img.shields.io/badge/GRU-7C3AED?style=flat-square) ![vLLM](https://img.shields.io/badge/vLLM-111827?style=flat-square) ![Qwen3-32B](https://img.shields.io/badge/Qwen3--32B-615CED?style=flat-square) ![MCP](https://img.shields.io/badge/MCP-111827?style=flat-square) |
| Infrastructure | ![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white) ![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white) ![VMware ESXi](https://img.shields.io/badge/VMware%20ESXi-607078?style=flat-square&logo=vmware&logoColor=white) ![pfSense](https://img.shields.io/badge/pfSense-212121?style=flat-square&logo=pfsense&logoColor=white) ![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white) ![MetalLB](https://img.shields.io/badge/MetalLB-0B7FAB?style=flat-square) ![NGINX Ingress](https://img.shields.io/badge/NGINX%20Ingress-009639?style=flat-square&logo=nginx&logoColor=white) |
| DevOps | ![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![Harbor](https://img.shields.io/badge/Harbor-60B932?style=flat-square&logo=harbor&logoColor=white) ![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat-square&logo=argo&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Trivy](https://img.shields.io/badge/Trivy-1904DA?style=flat-square&logo=trivy&logoColor=white) |
| Observability | ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white) ![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white) ![Loki](https://img.shields.io/badge/Loki-F46800?style=flat-square&logo=grafana&logoColor=white) ![Tempo](https://img.shields.io/badge/Tempo-F46800?style=flat-square&logo=grafana&logoColor=white) ![Alertmanager](https://img.shields.io/badge/Alertmanager-E6522C?style=flat-square&logo=prometheus&logoColor=white) ![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?style=flat-square&logo=opentelemetry&logoColor=white) ![Fluent Bit](https://img.shields.io/badge/Fluent%20Bit-49BDA5?style=flat-square&logo=fluentbit&logoColor=white) ![Cilium](https://img.shields.io/badge/Cilium-F8C517?style=flat-square&logo=cilium&logoColor=white) ![Hubble](https://img.shields.io/badge/Hubble-F8C517?style=flat-square) |

