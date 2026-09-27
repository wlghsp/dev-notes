# Observability Engineering Glossary 진도표
참고: Observability Engineering — Charity Majors, Liz Fong-Jones, George Miranda (O'Reilly, 2022)
원본 위치: assets/observability/Observability Engineering.pdf

전략: opentelemetry-study 스터디 예습이 목적이지만, 책 자체를 순서대로 읽는다.
스터디 회차와 겹치는 챕터는 opentelemetry-study/TRAINING_ROADMAP.md에서 별도로 표시해 확인한다.
진행 순서와 완료 여부는 챕터 종합 문서(chXX-*.md) 존재로 판단한다.

스터디 회차와의 대응 관계는 opentelemetry-study/TRAINING_ROADMAP.md 참고.

---

## 다음에 읽을 문서 (이 순서대로 진행)

핵심은 키워드 파일이다. 챕터 종합 문서(chXX-*.md)는 그 챕터에서 나온 키워드 파일들을 한 흐름으로 엮은 복습용이라, 키워드 파일들을 다 본 뒤 마지막에 훑는 용도다.

**Chapter 1** ✅
1. cardinality-dimensionality.md
2. known-unknown-vs-unknown-unknown.md
3. three-pillars.md
- (복습) ch01-what-is-observability.md

**Chapter 5** ✅
3. structured-event.md
- (복습) ch05-structured-events.md

**Chapter 6** ✅
4. trace-span.md
- (복습) ch06-stitching-events-into-traces.md

**Chapter 7** ✅
5. otel-instrumentation.md
- (복습) ch07-instrumentation-with-opentelemetry.md

**Chapter 2** ✅
6. institutional-knowledge-debugging.md
7. confirmation-bias-debugging.md
8. monitoring-debugging-failure-modes.md
- (복습) ch02-how-debugging-practices-differ.md

**Chapter 3** ✅
8. hero-culture-debugging.md
- (복습) ch03-lessons-from-scaling-without-observability.md

**Chapter 4 — How Observability Relates to DevOps, SRE, and Cloud Native** ⬜ (다음 차례)

**Chapter 8 — Analyzing Events to Achieve Observability** ⬜

**Chapter 9 — How Observability and Monitoring Come Together** ⬜

**Chapter 10~14 (Part III — Observability for Teams)** ⬜

**Chapter 15 — Build Versus Buy and Return on Investment** ⬜

**Chapter 16 — Efficient Data Storage** ⬜

**Chapter 17 — Cheap and Accurate Enough: Sampling** ⬜

**Chapter 18 — Telemetry Management with Pipelines** ⬜

**Chapter 19~22 (Part V — Spreading Observability Culture)** ⬜

---

책 목차 순서(1~22장) 그대로 읽는다. 스터디 회차와 특히 겹치는 챕터(1, 5~7, 16~18)는 위 목록에 표시돼 있고, 겹치는 정도는 opentelemetry-study/TRAINING_ROADMAP.md에서 확인한다.

---

## Part I — The Path to Observability

### Chapter 1 — What Is Observability? ✅
Observability의 수학적 정의, 소프트웨어 적용, Monitoring과의 차이.
스터디 0~1회차 예습과 직결.
생성된 문서: ch01-what-is-observability.md, cardinality-dimensionality.md, known-unknown-vs-unknown-unknown.md, three-pillars.md

### Chapter 2 — How Debugging Practices Differ Between Observability and Monitoring ✅
대시보드 기반 트러블슈팅의 한계, Observability가 디버깅을 어떻게 바꾸는가.
생성된 문서: ch02-how-debugging-practices-differ.md, institutional-knowledge-debugging.md, confirmation-bias-debugging.md, monitoring-debugging-failure-modes.md

### Chapter 3 — Lessons from Scaling Without Observability ✅
Parse의 스케일링 사례 연구. (스터디 회차와 직접 겹치는 챕터는 아님)
생성된 문서: ch03-lessons-from-scaling-without-observability.md, hero-culture-debugging.md

### Chapter 4 — How Observability Relates to DevOps, SRE, and Cloud Native
조직/문화 맥락. (스터디 회차와 직접 겹치는 챕터는 아님)

---

## Part II — Fundamentals of Observability

### Chapter 5 — Structured Events Are the Building Blocks of Observability ✅
Structured Events, 기존 Metric/Log의 한계.
스터디 1회차(Signal Model) 예습과 직결.
생성된 문서: ch05-structured-events.md, structured-event.md

### Chapter 6 — Stitching Events into Traces ✅
Trace의 구성 요소, Span, 수동 계측.
스터디 2~3회차(Instrumentation, Trace) 예습과 직결.
생성된 문서: ch06-stitching-events-into-traces.md, trace-span.md

### Chapter 7 — Instrumentation with OpenTelemetry ✅
OpenTelemetry Instrumentation 실습 코드 기반 설명. Automatic → Custom Instrumentation.
스터디 2회차(Zero-Code Instrumentation) 예습과 정확히 겹침.
생성된 문서: ch07-instrumentation-with-opentelemetry.md, otel-instrumentation.md

### Chapter 8 — Analyzing Events to Achieve Observability
Core Analysis Loop, AIOps에 대한 비판적 시각.

### Chapter 9 — How Observability and Monitoring Come Together
두 접근을 언제 같이 쓰는가.

---

## Part III — Observability for Teams
(10~14장. 조직 적용, SLO, 공급망. 스터디 회차와 직접 겹치는 챕터는 아님)

---

## Part IV — Observability at Scale

### Chapter 15 — Build Versus Buy and Return on Investment
(스터디 회차와 직접 겹치는 챕터는 아님)

### Chapter 16 — Efficient Data Storage
Time-Series Database가 Observability에 부적합한 이유, Honeycomb Retriever 사례.
스터디 6~7회차(TSDB 기초, Log 저장 전략) 예습과 정확히 겹침. 우선순위 높음.

### Chapter 17 — Cheap and Accurate Enough: Sampling
Sampling 전략. 5회차(Collector 파이프라인, tail sampling) 예습과 관련.

### Chapter 18 — Telemetry Management with Pipelines
Telemetry Pipeline의 속성과 운영. 4~5회차(Collector 구축/파이프라인) 예습과 직결.

---

## Part V — Spreading Observability Culture
(19~22장. 비즈니스 케이스, 조직 문화, 성숙도 모델. 스터디 회차와 직접 겹치는 챕터는 아님)

---

## 파일 관리 방식
키워드 파일은 glossary/books/observability-engineering/ 아래 개념별로 쌓는다.
챕터 종합 문서는 chXX-챕터제목.md 형식으로 같은 폴더에 쌓는다.
완료된 챕터만 아래에 실제 파일 목록으로 기록한다 (예상 키워드 미리 쓰지 않음).

## 완료된 챕터

- Chapter 1 — ch01-what-is-observability.md / cardinality-dimensionality.md, known-unknown-vs-unknown-unknown.md, three-pillars.md
- Chapter 5 — ch05-structured-events.md / structured-event.md
- Chapter 6 — ch06-stitching-events-into-traces.md / trace-span.md
- Chapter 7 — ch07-instrumentation-with-opentelemetry.md / otel-instrumentation.md
- Chapter 2 — ch02-how-debugging-practices-differ.md / institutional-knowledge-debugging.md, confirmation-bias-debugging.md, monitoring-debugging-failure-modes.md
- Chapter 3 — ch03-lessons-from-scaling-without-observability.md / hero-culture-debugging.md
