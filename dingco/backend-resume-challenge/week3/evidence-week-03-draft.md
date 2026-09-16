# Week 3 증거 초안

아래 내용을 대상 저장소의 `evidence/week-03__weekly-pr.md`에 옮긴다. `[대괄호]`는 최종 코드·테스트 결과에 맞게 채운다.

```markdown
# Week 3 증거

## 변경

- 측정 대상: studypass의 `GET /api/enrollments/search`, `GET /api/enrollments/stats`
- 변경한 이력서 문장: `resume/resume.md` 문장 3
- 개인정보·회사 기밀 제거 확인: 해당 없음
- search 개선:
  - `study_enrollment`에 `[schema.sql의 최종 인덱스명] (status, enrolled_at DESC, fee)` 복합 인덱스를 추가했다.
  - 목록·COUNT 쿼리가 모두 이 인덱스를 사용하도록 했다.
- stats 개선:
  - 원본 `study_enrollment` 집계 대신 `member_enrollment_fee_stats` 집계 테이블을 조회하도록 변경했다.
  - 기존 신청 데이터는 backfill로 최초 집계하고, 신규 신청은 동일 트랜잭션에서 집계 테이블도 갱신하도록 했다.
- 회귀 테스트:
  - `src/test/java/co/dingcodingco/studypass/enrollment/EnrollmentControllerTest.java`
  - search 조건·정렬·경계값과 stats의 회원별 count·totalFee·lastEnrolledAt을 검증한다.

## 검증

- 실행 환경: 로컬 macOS / Java 17 / Spring Boot 3.5 / MySQL 8.0 / Docker Compose
- 고정한 데이터·표본·명령:
  - 시드: members 2,000 / studies 300 / enrollments 200,000 / commentsPerStudy 30
  - search: `from=2026-08-01T00:00:00`, `to=2026-08-31T23:59:59`, `status=CONFIRMED`, `minFee=50000`
  - stats: `minFee=50000`
  - 워밍업 3회 후 HTTP 요청 5회 측정
  - 실행 계획: `EXPLAIN FORMAT=TREE`, 재구성 비교는 `EXPLAIN ANALYZE`

### search 개선 전후

| 항목 | 개선 전 | 개선 후 |
| --- | ---: | ---: |
| 목록 조회 접근 | `Table scan` / 199,580행 | `Index range scan` / 추정 10,612행 |
| COUNT 접근 | `Table scan` / 199,580행 | `Covering index range scan` / 추정 10,612행 |
| 목록 정렬 | `Sort: enrolled_at DESC` 존재 | 별도 Sort 제거 |
| API 평균 | 120.066ms | 14.653ms |
| API 중앙값 | 117.484ms | 11.043ms |

- 개선 전 API 5회: 130.773 / 118.893 / 116.113 / 117.484 / 117.067ms
- 개선 후 API 5회: 31.181 / 11.043 / 9.636 / 11.664 / 9.740ms
- 개선 후 목록은 `idx_enrollment_status_enrolled_at_fee`를 사용한 Index range scan으로 변경됐고, COUNT는 covering index range scan으로 변경됐다.

### stats 개선 전후

| 항목 | 개선 전 | 개선 후 |
| --- | ---: | ---: |
| 읽는 테이블 | `study_enrollment` | `member_enrollment_fee_stats` |
| 실제 스캔 행 수 | 200,000행 | 39,732행 |
| fee 필터 후 행 수 | 120,206행 | 23,835행 |
| DB 쿼리 실제 완료 시간 | 228ms | 31ms |
| DB 쿼리 시간 변화 | - | 약 86.4% 감소 |
| HTTP API 5회 | 전환 전 미기록 | 41.532 / 10.047 / 10.720 / 11.292 / 10.702ms |

- 개선 전 HTTP 응답 시간은 집계 테이블 전환 전에 기록하지 못했다.
- 대신 동일 MySQL·동일 원본 데이터·동일 `minFee=50000`에서 원본 집계 SQL과 집계 테이블 SQL을 `EXPLAIN ANALYZE`로 재측정했다.
- 개선 전은 `fk_enrollment_member` 인덱스를 200,000행 스캔했고, 개선 후는 집계 테이블 PRIMARY KEY를 39,732행 스캔했다.
- 개선 후에도 `ORDER BY enrollment_count DESC`를 위한 Sort는 남아 있다.

### 테스트

```text
[./gradlew test 실행 결과의 BUILD SUCCESSFUL 출력 붙여 넣기]
```

- 재현되지 않았거나 확인 불가인 내용:
  - stats 개선 전 HTTP API 5회 시간은 전환 전에 기록하지 못해 확인 불가다.
  - 전후 비교는 동일 조건의 `EXPLAIN ANALYZE` DB 쿼리 시간으로 보완했다.

## 선택 근거

- 비교한 대안:
  - search: `(status, fee, enrolled_at)` / `(status, enrolled_at, fee)`
  - stats: 원본 테이블 인덱스 추가 / 집계 테이블 / 원본 테이블 반정규화
- 선택한 이유:
  - search는 등치 조건인 `status`를 먼저 고정하고, 범위 조건이자 정렬 기준인 `enrolled_at`을 다음에 둬 최신순 조회와 날짜 범위 탐색을 함께 처리하려 했다.
  - `fee`는 두 번째 범위 조건이므로 후속 필터로 사용했다.
  - stats는 전체 원본 데이터를 매번 회원별로 집계하는 비용이 커, 회원·금액별 사전 집계 테이블로 원본 200,000행을 39,732행으로 줄였다.
- 버린 대안이 더 적합해지는 조건:
  - fee가 날짜보다 훨씬 선택적이고 최신순 정렬 요구가 없다면 `(status, fee, enrolled_at)`이 유리할 수 있다.
  - 데이터 갱신이 매우 많고 통계 조회가 드물다면 집계 테이블 유지 비용 때문에 원본 집계 또는 다른 반정규화 전략이 더 적합할 수 있다.

## 근거형 질문

### 질문 1

질문: 현재 쿼리에서 병목(성능 저하) 가능성이 있을 만한 부분은 어디라고 생각하나요?

- 최초 판단: search는 조건용 인덱스가 없어 풀 스캔과 정렬이 병목이고, stats는 원본 전체 그룹 집계가 병목이다.
- 연결한 저장소 안 코드·로그·결과:
  - `EnrollmentRepository.search`, `countSearch`, `statsByMember`
  - search 개선 전 EXPLAIN의 `Table scan` 199,580행
  - stats 개선 전 `EXPLAIN ANALYZE`의 원본 200,000행 스캔
- 검증 후 답변: search는 목록과 COUNT가 모두 전체 테이블을 읽었고 목록에는 Sort까지 발생했다. stats는 fee 조건 뒤 회원별 GROUP BY와 정렬을 위해 원본 200,000행을 읽는 비용이 확인됐다.

### 질문 2

질문: 인덱스(또는 복합 인덱스)를 새로 추가한다면, 어떤 컬럼(들)에 추가하고 싶나요?

- 최초 판단: search에는 `(status, enrolled_at DESC, fee)` 복합 인덱스를 추가하고, stats는 집계 테이블을 적용한다.
- 연결한 저장소 안 코드·로그·결과:
  - `src/main/resources/schema.sql`
  - search 개선 후 Index range scan 및 Covering index range scan
  - `member_enrollment_fee_stats` 스키마와 조회 코드
- 검증 후 답변: search는 status 등치 조건과 날짜 범위·정렬을 우선 처리하도록 `(status, enrolled_at DESC, fee)` 복합 인덱스를 선택했다. stats는 원본 `study_enrollment`에 인덱스를 추가하는 대안도 검토했지만, 회원별 GROUP BY 자체의 비용은 인덱스만으로 제거되지 않아 인덱스 대신 집계 테이블을 선택했다.

### 질문 3

질문: 인덱스를 적용하거나 쿼리를 튜닝한 뒤, 성능을 어떻게 측정·검증할 계획인가요?

- 최초 판단: 동일 시드·파라미터·워밍업 조건에서 EXPLAIN과 API 시간을 비교하고, 회귀 테스트로 결과 동일성을 확인한다.
- 연결한 저장소 안 코드·로그·결과:
  - search API 5회 전후 측정값
  - stats 원본·집계 SQL의 `EXPLAIN ANALYZE`
  - `EnrollmentControllerTest`
- 검증 후 답변: search는 동일 조건 API 평균이 120.066ms에서 14.653ms로 감소했다. stats는 동일 조건 DB 쿼리 실제 완료 시간이 228ms에서 31ms로 감소했다. API 결과 계약은 회귀 테스트로 고정했다.

### 질문 4

질문: 만약 인덱스 적용 후 데이터 삽입·수정 시 성능 저하가 생긴다면, 어떻게 대처할 수 있을까요?

- 최초 판단: 인덱스와 집계 테이블은 읽기를 빠르게 하지만 INSERT·UPDATE 때 유지 비용을 추가한다.
- 연결한 저장소 안 코드·로그·결과:
  - `EnrollmentService.enroll()`
  - `MemberEnrollmentStatsRepository.increment()`
  - `SeedRunner.backfillEnrollmentStats()`
- 검증 후 답변: 신규 신청 시 원본 신청 저장과 집계 갱신이 함께 수행되므로 쓰기 비용과 정합성 관리가 추가된다. 현재 취소·수정 API는 없지만, 추가될 경우 count·totalFee·lastEnrolledAt 보정 로직도 함께 구현해야 한다. 만약 쓰기 저하가 실제로 확인되면, search 인덱스는 실제 조회 조건에 안 쓰이는 컬럼이 포함돼 있지 않은지 먼저 확인해 불필요한 인덱스부터 제거하고, stats 집계 테이블은 매 신청마다 즉시 갱신하는 대신 배치나 비동기 갱신으로 바꿔 쓰기 경로의 즉시 비용을 줄이는 방향을 검토할 것이다.

## 리뷰 반영

- 최초 제출: 자동 리뷰 수신 전
- 재제출: [리뷰 수신 후 지적 내용과 보완 파일을 기록]
```

제출 전에 `schema.sql`의 인덱스명과 실제 `SHOW INDEX` 결과가 하나로 일치하는지 확인한다.
