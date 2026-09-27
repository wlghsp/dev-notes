# OpenTelemetry 스터디 예습 로드맵

> 스터디가 이미 짜놓은 커리큘럼(README.md, curriculum.png)을 따라가며, 각 회차 시작 전에 낯설지 않을 정도로 미리 훑어두기 위한 로드맵이다.
> 회차를 "완료"하거나 "테스트 통과"하는 트레이닝이 아니라 예습용이므로, 이해도 테스트나 Phase 완료 기준은 두지 않는다.
> glossary 파일 생성은 지호님이 요청할 때 진행한다. Claude가 먼저 만들지 않는다.

---

## 예습 진행 현황

- 0~1회차 대비 (Observability, Signal Model): ✅ observability.md, signal.md 작성 완료
- 2회차 대비 (Instrumentation): ⬜
- 3회차 대비 (Context Propagation, W3C Trace Context): ⬜
- 4~5회차 대비 (Collector 구조): ⬜
- 6~7회차 대비 (TSDB, Log 저장소): ⬜
- 8회차 대비 (종합 설계): 별도 예습 불필요 — 앞선 회차 내용을 엮는 회차

---

## 회차별 예습 후보 개념

### 0회차 — OT, OpenTelemetry, Observability 기본
Observability, 기존 APM(Pinpoint, Datadog, Elastic APM, AWS CloudWatch)이 무엇인지 이름 정도만 알아둔다.
**작성된 glossary 파일**: observability.md

### 1회차 — OpenTelemetry 아키텍처 & OLTP 데이터 모델
Signal(Trace/Metric/Log), API/SDK/Collector 역할 구분. 이후 회차 전체의 뼈대가 되는 부분이라 우선순위가 높다.
**작성된 glossary 파일**: signal.md

### 2회차 — Zero-Code Instrumentation
Instrumentation의 의미, Java Agent가 바이트코드 조작으로 계측하는 개념, correlation.
**작성된 glossary 파일**: (없음)

### 3회차 — Trace 심화, Context Propagation
Context Propagation, W3C Trace Context(traceparent/tracestate), 비동기 처리에서 context가 유실되는 이유. 커리큘럼 상 가장 난이도가 높을 것으로 예상되는 회차라 미리 정리해두면 도움이 큼.
**작성된 glossary 파일**: (없음)

### 4~5회차 — Collector 구조
Receiver/Processor/Exporter 파이프라인, Backpressure.
**작성된 glossary 파일**: (없음)

### 6~7회차 — 저장소
Time-Series DB가 일반 RDB와 다른 점, Cold Storage.
**작성된 glossary 파일**: (없음)

---

## 참고
- 모든 glossary 산출물은 glossary/topics/observability/ 아래 개념별 단독 파일로 쌓인다
- 이 로드맵은 예습 진행 상황 추적용이고, 스터디 커리큘럼 원문은 README.md/curriculum.png를 기준으로 한다
- 실제 회차 진행 기록(숙제, 인증샷, 그날의 질문)은 스터디 진행 이후 README.md의 "회차별 기록" 섹션에 남긴다
