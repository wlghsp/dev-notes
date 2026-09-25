# Week 5 증거 초안

아래 내용을 대상 저장소의 `evidence/week-05__weekly-pr.md`에 옮긴다.

```markdown
# Week 5 증거

## 변경

- 대상 API: studypass의 `GET /api/studies/{studyId}/comments`
- 변경한 이력서 문장: `resume/resume.md` 문장 6
- 개인정보·회사 기밀 제거 확인: 해당 없음
- 선택한 항목: N+1 (댓글 목록에서 작성자 조회). 벌크 연산·Stream 필터 오버헤드·비동기 처리는 이 프로젝트 조회 패턴에 적용 여지가 없어 제외했다.
- 선택한 기법: fetch join. `CommentRepository`에 `findByStudyIdWithMemberOrderByIdAsc`(JPQL `join fetch`)를 추가하고 `StudyQueryService.comments()`가 이 메서드를 쓰도록 교체했다. 기존 `findByStudyIdOrderByIdAsc`는 회귀 테스트의 "개선 전" 비교 기준으로 남겼다.
- 측정 장치: `QueryCountProbe`(`src/main/java/co/dingcodingco/studypass/support/QueryCountProbe.java`) — Hibernate `Statistics.getPrepareStatementCount()`를 감싼 것. 테스트/로컬 전용으로 두고 상시 프로덕션 노출용 엔드포인트는 만들지 않았다.
- 선택 확장: Grafana 대시보드로 지표 시각화. `hibernate_statements_total{status="prepared"}`을 `/actuator/prometheus`에 노출시키고, 기존 Prometheus·Grafana(2주차부터 `docker-compose.yml`에 있음)에 쿼리 수·응답 시간 패널을 추가했다.
- 회귀 테스트: `CommentQueryTest.java` — 응답 계약 동일성 1개, 쿼리 수 상한 검증 1개.

## 검증

- 환경: 로컬 macOS / Java 17 / Spring Boot 3.5 / MySQL 8.0(도커) / 테스트는 H2(`MODE=MySQL`)
- 고정 조건: studyId=1(시드 기준 댓글 30건), 동일 studyId로 개선 전/후 측정

### 개선 전/후 비교

```text
쿼리 수:   31건 → 1건    (약 31배 감소)
응답 시간: 178ms → 13.7ms (약 13배 감소, 같은 방향으로 개선)
```

개선 전 SQL 로그상 `study_comment` 조회 1건 이후 `member ... where m1_0.id=?`가 30회 반복됐고(31건), prep-questions 5번 예측과 정확히 일치했다. 개선 후에는 `join fetch`로 select 1건만 실행됐다.

### 회귀 테스트 (`CommentQueryTest`)

```text
BUILD SUCCESSFUL
commentsResponseMatchesLazyLoadedResult PASSED — 응답 계약(개수·id·content·author) 동일성 확인
commentsQueryCountIsLessThanCommentCount PASSED — 개선 후 쿼리 수(1건) < 댓글 수(5건), N+1 해소 확인
```

쿼리 수는 절대값(`=1`)이 아니라 "댓글 수보다 적다"는 상한으로 assert했다(fetch join 내부 쿼리 형태가 바뀌어도 테스트가 과도하게 깨지지 않게 하려는 판단).

### Grafana 대시보드 (선택 확장)

개선 후(fetch join) 상태에서 호출 시 `hibernate_statements_total{status="prepared"}`이 호출당 1건씩(7→11) 작게 증가했고, 코드를 N+1 버전으로 잠깐 되돌려 호출하면 호출당 31건씩(0→93) 크게 증가했다. 같은 패널에서 두 계단 크기가 뚜렷이 대비되어 curl+SQL 로그 수치와 같은 방향임을 재확인했다. 캡처 후 코드는 fetch join으로 원상복구했다.

## 겪은 문제

- `hibernate.generate_statistics: true`만으로는 `/actuator/prometheus`에 `hibernate_*` 메트릭이 뜨지 않았다. `session_factory.name` 미설정과, Spring Boot 3 + Hibernate 6에서 필요한 `hibernate-micrometer` 의존성 누락이 원인이었다. 둘 다 추가해 해결했다.
- 회귀 테스트에 `@Transactional`을 걸었더니 `TestRestTemplate`의 실제 HTTP 요청이 별도 커넥션에서 처리되어 아직 커밋 안 된 `@BeforeEach` 데이터를 못 봤다. `@Transactional`을 빼고 `expected` 조회를 fetch join 메서드로 바꿔 해결했다.
- `@BeforeEach`의 고정 이메일 문자열이 테스트 간 누적되며 unique 제약 위반이 났다. `UUID.randomUUID()`로 유일화해 해결했다(`EnrollmentConcurrencyTest`와 같은 패턴).

## 리뷰 반영

- 최초 제출: 자동 리뷰 수신 전
- 재제출: [리뷰 수신 후 지적 내용과 보완 파일을 기록]
```
