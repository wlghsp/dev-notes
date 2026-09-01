# Week 1 증거 (초안)

studypass 저장소의 evidence/week-01__weekly-pr.md에 옮겨 적을 초안.

## 변경

- 측정 대상: 이 저장소의 studypass (기본)
- 변경한 이력서 문장: resume/resume.md 문장 1
- 개인정보·회사 기밀 제거 확인: 해당 없음 (측정값·코드 근거만 사용)

## 검증

- 실행 환경: MacBook Pro M3 16GB, docker compose(db)로 MySQL 8.0 로컬 기동, `gradle bootRun`으로 애플리케이션 실행
- 고정한 데이터·표본·명령:
  - 시드 규모: members 2000 / studies 300 / enrollments 200000 / commentsPerStudy 30 (`app.seed.*`)
  - 측정 대상: `GET /api/studies?page=0&size=20`
  - 측정 명령: `curl -s -o /dev/null -w "요청 %{time_total}s\n" "..."` (워밍업 1회 제외, 3회 측정)
- 실행 결과:
  - 응답 시간 3회: 0.025574s / 0.012112s / 0.017677s
  - SQL 로그: `WHERE` 절 없이 `LIMIT ?, ?` 쿼리와 `count(s1_0.id)` 쿼리를 각각 실행

    ```
        select
            s1_0.id,
            s1_0.capacity,
            s1_0.category,
            s1_0.created_at,
            s1_0.enrolled_count,
            s1_0.fee,
            s1_0.opened_at,
            s1_0.status,
            s1_0.title
        from
            study s1_0
        limit
            ?, ?
    2026-08-28T10:26:21.632+09:00 TRACE 850 --- [nio-8080-exec-1] org.hibernate.orm.jdbc.bind : binding parameter (1:INTEGER) <- [1036]
    2026-08-28T10:26:21.633+09:00 TRACE 850 --- [nio-8080-exec-1] org.hibernate.orm.jdbc.bind : binding parameter (2:INTEGER) <- [74]
    2026-08-28T10:26:21.648+09:00 DEBUG 850 --- [nio-8080-exec-1] org.hibernate.SQL :
        select
            count(s1_0.id)
        from
            study s1_0
    ```
  - 병목 예측: 두 쿼리 모두 조건절이 없어 스캔 범위를 못 좁힌다. 데이터가 늘면 스캔 비용이, 동시 요청이 늘면 그 비용이 요청 수만큼 반복된다. `study`에는 PK 외 인덱스가 없지만 조건절 자체가 없어 지금은 인덱스를 추가해도 실행계획이 안 바뀐다 (인덱스는 3주차 대상)
  - 회귀 테스트: `src/test/java/co/dingcodingco/studypass/study/StudyControllerTest.java`
    - `listReturnsOkWithExpectedFields`: 상태 200, 응답 필드(totalElements/content 및 항목 필드) 존재 확인
    - `sizeParameterIsCappedAt100`: 105건 저장 후 size=500 요청 시 content가 100건으로 캡되는지 확인
- 재현되지 않았거나 확인 불가인 내용:
  - 동시 요청 부하 상태의 응답 시간 — 확인 불가, 2주차 k6로 측정 예정
  - 대규모 데이터(수만 건 이상)에서의 실측 응답 시간 — 확인 불가

## 선택 근거

- 비교한 대안: 해당 없음 (선택 확장 미진행)
- 선택한 이유:
- 버린 대안이 더 적합해지는 조건:

## 근거형 질문

### 1. 이 기능에서 사용자가 늘면 가장 먼저 무엇이 병목이 될 것 같고, 그 근거는 무엇인가요?

- 최초 판단: `count(*)` 쿼리의 전체 스캔이 먼저 병목이 된다. 조건절이 없어 데이터가 늘수록 스캔 비용이 커지고, 동시 요청이 늘수록 그 비용이 반복된다. 뒷페이지로 갈수록 OFFSET 비용도 함께 늘어난다.
- 연결한 저장소 안 코드·로그·결과: `StudyController.list()`의 `findAll(PageRequest...)` 호출, SQL 로그의 `count(s1_0.id)`/`limit ?, ?` 쿼리, schema.sql의 `study` 테이블(PK 외 인덱스 없음)
- 검증 후 답변: 현재 300건에서는 12~26ms로 병목이 안 드러난다. 예측이 맞는지는 데이터를 늘리거나 EXPLAIN으로 확인해야 하며, 이번 주차는 예측과 코드 근거까지만 확인했다.

### 2. 측정한 응답 시간이 실제 사용 패턴을 얼마나 대표하며 어떤 한계가 있나요?

- 최초 판단: 대표성이 낮다. 순차 요청 3회, 단일 사용자, 고정 조건(page=0&size=20)만 측정해 동시 트래픽이나 다른 페이지 요청 패턴을 반영하지 못한다.
- 연결한 저장소 안 코드·로그·결과: 측정 명령과 3회 결과, 시드 규모(studies 300건) — evidence "## 검증" 항목
- 검증 후 답변: "동시 요청 없이, 데이터가 적고, 첫 페이지만 조회했을 때"만 대표한다. 동시 요청·뒷페이지·대규모 데이터는 대표하지 못하며, 2주차 k6로 보완할 계획이다.

### 3. 작성한 이력서 문장에서 팀의 성과와 본인의 기여를 어떤 증거로 구분할 수 있나요?

- 최초 판단: 1주차 시점엔 구분이 성립하지 않는다. 개인 실습 프로젝트라 팀 성과라는 개념 자체가 없고, 문장 1의 행동은 전부 본인이 직접 했다.
- 연결한 저장소 안 코드·로그·결과: 없음 (비교 대상 부재)
- 검증 후 답변: 이 질문은 실무 팀 프로젝트에서 성립하는 질문이며, 이 저장소에는 해당하지 않는다. 실제로 구분하려면 커밋 기록·PR 리뷰·담당 모듈 범위 같은 근거가 필요하다.

### 4. 현재 문장에서 아직 검증할 수 없는 주장과 이를 보완할 계획은 무엇인가요?

- 최초 판단: "병목이 count 쿼리다"라는 예측 자체가 미검증이다. SQL 로그로 어떤 쿼리가 나가는지만 확인했고, 실제로 느려지는지는 측정하지 않았다.
- 연결한 저장소 안 코드·로그·결과: evidence "## 검증"의 병목 예측 항목, resume.md 문장 1의 "한계"
- 검증 후 답변: 2주차 k6로 동시 부하 상태의 실제 영향을 측정하고, 3주차 EXPLAIN으로 실행계획을 직접 확인해 검증한다. 그전까지는 "병목 예측"을 사실이 아닌 예측으로 표기한다.

## 리뷰 반영

- 최초 제출: 자동 리뷰 수신 전
- 재제출: 지적 내용과 보완한 파일을 기록
