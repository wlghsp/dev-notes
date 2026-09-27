# OpenTelemetry Monitoring 스터디

## 스터디 소개

### 배경
새로운 부서로 오면서 Observability Platform 실무 리딩을 맡게 되었고, 이 과정에서 OpenTelemetry와 모니터링 플랫폼 관련 조직 내 세미나를 진행하게 되었다. OpenTelemetry가 최근 관측 도구의 사실상 표준이 되어가는데, 한국 내 관련 커뮤니티가 아직 많지 않아 기술적 공감대 형성을 위해 조직 밖에서도 세미나를 열게 되었다.

### 어떤 사람에게 도움이 되는 스터디인지
- 서비스를 운영하면서 로그, CPU 사용량, Stack Trace 등을 뒤져본 적이 있는 사람
- 회사에서 Datadog이나 유사 도구를 이미 써본 사람
- Observability에 대해 궁금한 사람

## 스터디 안내

- 진행 기간: 10월 초 논의 예정, 8회차
- 진행 요일/시간: 논의 예정, 회차당 1.5~2시간
- 진행 장소: 온라인
- 모집 인원: 주최자 포함 9명 (최대 8명)

## 진행 방식

- 미팅 전: 별도 준비 없음. 추천 도서 내용을 미리 봐도 좋음
- 미팅 중: 세미나 형식. 자료 및 실습 환경 세팅은 사전 제공
- 미팅 후: 숙제 진행 후 다음 회차 전까지 인증샷 업로드

### 사용 도구
관측 대상 애플리케이션은 Spring Boot. OpenTelemetry, Kafka(공용 클러스터 제공, 깊게 알 필요 없음), OpenSearch 또는 Grafana Loki, Tempo, VictoriaMetrics. 위 도구를 몰라도 스터디 참여에는 문제없다.

## 커리큘럼

### 0회차 — OT, OpenTelemetry, Observability 기본
커리큘럼 소개, Observability 개념, 기존 APM 둘러보기(Pinpoint, Datadog, Elastic APM, AWS CloudWatch 등). 이날은 짧게 진행 예정.

### 1회차 — OpenTelemetry 아키텍처 & OLTP 데이터 모델
기본적인 OTel 설명, Signal Model(Trace, Metric, Log), API/SDK/Collector 구조. 기본 환경 세팅을 통한 OLTP 확인. 실습 환경용 Docker Compose 제공.

### 2회차 — OpenTelemetry Zero-Code Instrumentation
Java Agent/Spring Boot Starter를 통한 계측 실습. 기본 파이프라인 구축, trace/metric 자동 생성 확인, correlation 연동. Metric은 Prometheus, Log는 Grafana Loki, Trace는 Grafana Tempo 사용.

### 3회차 — Trace 좀 더 파보기: 수동 계측 & 분산 Trace & Context Propagation
API/SDK 직접 사용을 통한 Trace 계측 커스터마이징. W3C Trace Context(traceparent/tracestate), propagator 동작 원리. HTTP/Kafka 경계 관통, 비동기 처리 시 context 유실 등.

### 4회차 — Collector 구축
Agent Pattern/Gateway Pattern. Receiver/Processor/Exporter/Extension 등 Collector 구성 요소.

### 5회차 — Collector 파이프라인 심화
Processor 체인 설계(filter/attributes/transform/tail sampling/memory_limiter/connector 등). Back Pressure, Error Propagation.

### 6회차 — Metric: TSDB 기초, 플랫폼 고르기
Time-Series DB의 특성과 내부 구조. Prometheus, VictoriaMetrics 살펴보기.

### 7회차 — Log: 어디에 데이터를 넣을 것인가
Loki의 아쉬운 부분. OpenSearch에 넣는다면? Cold Storage 등의 이슈.

### 8회차 — 종합 플랫폼 설계
전체 파이프라인을 직접 만들어보기. 추가 이슈 논의.

> 📷 원본 커리큘럼 표: curriculum.png 참고

## 진행 현황

- 0회차: ⬜ 진행 전

## 미리 학습해두면 좋은 개념 (glossary/topics/observability/)

- observability.md — Monitoring과의 차이, Trace/Metric/Log 세 축
- signal.md — OTel이 세 데이터를 signal로 묶은 이유, correlation과의 관계

스터디 진행하면서 새로 나오는 개념(Context Propagation, W3C Trace Context, Instrumentation, Collector 파이프라인, TSDB 구조 등)은 그때그때 glossary/topics/observability/에 단독 파일로 추가한다.

## 회차별 기록

회차가 끝나면 이 아래에 `## N회차 기록`으로 이어서 남긴다. 숙제 진행 내용, 인증샷 캡처 위치, 그날 정리한 질문 등을 기록한다.
