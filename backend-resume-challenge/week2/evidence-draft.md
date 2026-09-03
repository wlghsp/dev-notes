# Week 2 evidence 초안

studypass의 evidence/week-02__weekly-pr.md에 옮겨 적을 내용. execution-guide.md의 1~6단계 결과를 근거형 질문 형식(week-01__weekly-pr.md와 동일한 틀)에 맞춰 정리.

---

## 변경

- 측정 대상: 이 저장소의 studypass / GET /api/studies (1주차와 동일)
- 변경한 이력서 문장: resume/resume.md 문장 1(한계 교체), 문장 2(신규)
- 개인정보·회사 기밀 제거 확인: 해당 없음

## 검증

- 실행 환경: (1주차와 동일 환경인지 확인 후 기입)
- 고정한 데이터·표본·명령:
  - 시드 규모: members 2000 / studies 300 / enrollments 200000 / commentsPerStudy 30 (1주차와 동일)
  - Prometheus·Grafana: docker compose로 함께 기동, `/actuator/prometheus`에서 지표 수집 확인
  - k6 스크립트: `k6-scripts/baseline.js` (10→30 VU, 총 2분)
- Prometheus·Grafana 수집 지표 (execution-guide.md 2단계):
  - (2-3, 2-5 수치 옮기기: system_cpu_usage, process_cpu_usage, jvm_memory_used/max_bytes)
- k6 기준선 TPS·응답 시간 분포 (execution-guide.md 3단계, 개선 전):
  - (3-3 수치 옮기기: TPS, 평균, p90, p95, p99, http_req_failed)
- 부하 중 병목 판단 (execution-guide.md 4단계):
  - (4-1, 4-3 수치와 판단 근거 옮기기: CPU가 가장 먼저 반응, 메모리·커넥션 풀은 여유)
- 적용한 개선 (execution-guide.md 5단계):
  - (선택한 개선 방법, 변경한 src/main·src/test 파일 옮기기)
- 개선 후 재측정 결과 (execution-guide.md 6단계):
  - (6-1, 6-2 수치 옮기기: 개선 전후 비교표)
  - 참고: 재측정을 두 번 실행했고 첫 실행은 워밍업 부족으로 편차가 컸음(JIT 컴파일·커넥션 풀 워밍업 추정) — 재실행 값을 최종 결과로 사용
- 재현되지 않았거나 확인 불가인 내용:
  - (있으면 기입, 없으면 "해당 없음")

## 선택 근거

- 비교한 대안: OFFSET 페이징 유지 + count 쿼리만 최적화 / Page→Slice 전환 / 커서 기반 페이징
- 선택한 이유: (improvement-guide.md "왜 커서 페이징을 선택했는가" 옮기기 — PK 인덱스 바로 활용 가능, 기존 회귀 테스트 영향 최소화, count 문제는 API 계약을 더 크게 바꾸므로 범위 밖)
- 버린 대안이 더 적합해지는 조건: 임의 페이지 점프(page=5)나 총 개수(totalElements) 표시가 요구사항에 반드시 필요한 경우

## 근거형 질문

### 1. 현재 프로젝트에 사용자가 400명 이상으로 증가하면 어떤 문제가 발생할까요?
- 최초 판단: CPU가 가장 먼저 한계에 닿을 것이다. 400명은 이번에 측정한 최대 부하(30 VU) 대비 약 13배 규모다.
- 연결한 저장소 안 코드·로그·결과: execution-guide.md 4-1(system_cpu_usage 부하 전 mean 0.343 → 부하 중 max 0.655, process_cpu_usage는 mean 0.00949 → max 0.189로 약 20배 상승), 4-3(메모리·MySQL 커넥션 풀은 이번 부하로는 한계에 안 닿음)
- 검증 후 답변: 30 VU에서 이미 CPU가 가장 뚜렷하게 반응했고 메모리·커넥션 풀은 여유가 있었으므로, 400명 규모에서도 먼저 포화되는 자원은 CPU일 가능성이 높다. 다만 정확히 몇 명에서 CPU가 100%에 닿는지, 그 시점에 응답 시간이 얼마나 나빠지는지는 이번 측정(최대 30 VU)만으로는 특정할 수 없다 — 실제로 400 VU까지 올려 재현하지는 않았고, 30 VU 결과로부터의 추론까지만 확인했다.

### 2. 현재 제공하던 기능에서 추가로 제공될만한 기능이 있다면 뭐가 있을까요? 해당 기능을 제공하려면 어떤 변화가 필요할까요?
- 최초 판단: 스터디 목록을 조건별로 좁혀 찾는 검색/필터 기능(카테고리, 모집 상태, 기간 등)이 다음 후보다. 지금은 카테고리 단순 조회(`findByCategory`)와 인기순 조회만 있고 복합 조건 조회가 없다.
- 연결한 저장소 안 코드·로그·결과: `StudyRepository`의 기존 메서드(`findByCategory`, `findTop10ByStatusOrderByEnrolledCountDesc`), 이번 주차 커서 페이징 추가 시 src/main(Repository·Controller)과 src/test를 함께 변경한 패턴
- 검증 후 답변: 복합 조건 검색을 추가하면 이번 주차처럼 Repository에 쿼리 메서드(또는 `@Query`) 추가, Controller에 파라미터 확장, 회귀 테스트 추가가 함께 필요하다. 다만 조건이 늘어날수록 인덱스 설계가 함께 필요해지는데, 이는 3주차(인덱스와 쿼리 플랜) 영역과 바로 연결된다 — 즉 기능 하나를 늘릴 때마다 "코드 변경"과 "인덱스 설계"가 세트로 필요하다는 걸 이번 주차 개선 경험으로 확인했다.

### 3. 현재 프로젝트에 예상보다 많은 사용자가 유입된다면 저희는 어떤걸 공부해야 할까요?
- 최초 판단: 이번 주차 병목 판단에서 "CPU가 먼저 반응한다"까지는 확인했지만 "정확히 어느 쿼리·어느 코드 경로가 그 CPU를 쓰는가"는 확인하지 못했다. 이 공백을 메우는 지식이 다음에 필요하다.
- 연결한 저장소 안 코드·로그·결과: execution-guide.md 4-3의 한계 서술("정확히 어느 쿼리가 CPU 상승의 원인인지까지는 이번 측정만으로는 특정하지 못했다 — EXPLAIN까지 확인해야 하는 3주차 영역과 연결")
- 검증 후 답변: 실행계획(EXPLAIN)과 인덱싱 — 어떤 쿼리가 풀스캔을 하는지 직접 확인하는 능력. 이번 주차에 커넥션 풀은 여유가 있었지만, 사용자가 더 늘면 HikariCP 풀 크기·스레드 모델(톰캣 요청 처리 방식)도 같이 공부해야 한다. 그리고 지금은 캐싱을 전혀 안 쓰고 있어, 반복 조회가 많은 `GET /api/studies/popular` 같은 엔드포인트는 캐싱 전략(Redis는 이미 docker compose에 떠 있음)도 다음 학습 대상이다.

## 리뷰 반영

자동 리뷰 수신 전
