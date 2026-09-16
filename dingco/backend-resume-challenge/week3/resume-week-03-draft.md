# Week 3 resume.md 문장 3·4 초안

대상 저장소의 `resume/resume.md` 문장 3·4에 아래 내용을 직접 옮겨 적는다. search 복합 인덱스와 stats 집계 테이블은 문제·측정값·트레이드오프가 다르므로 하나의 성과 문장으로 합치지 않는다.

```markdown
### 문장 3

- 문장: 20만 건 신청 데이터의 search 실행 계획에서 목록·COUNT 쿼리 모두 전체 스캔하는 것을 확인한 뒤 `(status, enrolled_at DESC, fee)` 복합 인덱스를 적용해, 동일 조건 API 평균 응답 시간을 120.066ms에서 14.653ms로 약 87.8% 낮췄다.
- 본인 행동: 목록 SELECT와 COUNT SELECT의 EXPLAIN을 분리해 분석하고, 등치 조건인 status·범위와 정렬 조건인 enrolledAt·후속 fee 필터 순으로 복합 인덱스를 설계했다. 기간·status·최소 금액·최신순 정렬을 검증하는 search 회귀 테스트를 추가했다.
- 전후 조건과 결과:
  - search: members 2,000 / studies 300 / enrollments 200,000, `from=2026-08-01T00:00:00`, `to=2026-08-31T23:59:59`, `status=CONFIRMED`, `minFee=50000`, 워밍업 3회 후 5회 측정
  - search 평균: 120.066ms → 14.653ms, 중앙값: 117.484ms → 11.043ms
- 저장소 안 근거:
  - 측정 결과·EXPLAIN: evidence/week-03__weekly-pr.md
  - 인덱스 DDL: src/main/resources/schema.sql
  - 회귀 테스트: src/test/java/co/dingcodingco/studypass/enrollment/EnrollmentControllerTest.java
- 한계: 현재 시드 규모와 한 달 검색 조건에서의 로컬 측정이며, 실제 데이터 분포·기간 조건이 달라지면 최적 컬럼 순서도 달라질 수 있다.

### 문장 4

- 문장: 회원별 신청 통계가 원본 신청 20만 건을 매번 집계하는 비용을 확인한 뒤 회원·금액별 집계 테이블을 적용해, 동일 조건 DB 쿼리 실제 완료 시간을 228ms에서 31ms로 약 86.4% 줄이고 실제 스캔 행 수를 200,000행에서 39,732행으로 줄였다.
- 본인 행동: minFee 조건을 유지하기 위해 회원 단위 단일 합계가 아닌 회원·금액 단위 집계 테이블을 설계했다. 기존 신청은 백필하고, 신규 신청은 원본 저장과 같은 트랜잭션에서 집계값을 갱신하도록 연결했다. 회원별 count·totalFee·lastEnrolledAt을 검증하는 stats 회귀 테스트를 추가했다.
- 전후 조건과 결과:
  - stats: 동일 MySQL·동일 원본 데이터·`minFee=50000`에서 `EXPLAIN ANALYZE` 비교
  - DB 쿼리 실제 완료 시간: 228ms → 31ms
  - 실제 스캔 행 수: 200,000행 → 39,732행
- 저장소 안 근거:
  - 측정 결과·EXPLAIN: evidence/week-03__weekly-pr.md
  - 집계 테이블 DDL: src/main/resources/schema.sql
  - stats 조회·신규 신청 집계 갱신: src/main/java/co/dingcodingco/studypass/enrollment/
  - 백필: src/main/java/co/dingcodingco/studypass/seed/SeedRunner.java
  - 회귀 테스트: src/test/java/co/dingcodingco/studypass/enrollment/EnrollmentControllerTest.java
- 한계: stats 개선 전 HTTP API 5회 응답 시간은 전환 전에 기록하지 못했다. 따라서 전후 수치는 HTTP 응답 시간이 아니라 동일 조건의 DB 쿼리 `EXPLAIN ANALYZE` 결과다. 현재 집계 테이블도 fee 필터와 집계 후 정렬 비용은 남아 있어 추가 인덱스 검토가 필요하다.
```
