# 5주차 준비: 알아야 할 것들

missions/README.md Week 5("댓글 목록의 쿼리 수를 세고 N+1·벌크·비동기 중 하나를 개선하기")를 실제로 하기 전에 먼저 답할 수 있어야 하는 질문들. studypass 저장소(challenge-backend-resume-2026-08-wlghsp-r17) 코드를 근거로 채운다.

---

## 1. N+1 지점 이해

- `StudyQueryService.comments(studyId)`는 어떤 순서로 동작하는가? (`commentRepository.findByStudyIdOrderByIdAsc(studyId)`로 댓글 목록 조회 → 각 댓글을 stream으로 돌며 `comment.getMember().getNickname()` 호출)

1. commentRepository.findByStudyIdOrderByIdAsc(studyId); 댓글 목록 조회
2. 각 댓글을 stream 으로 돌며 comment.getMember().getNickname() 호출

- `Comment.member`는 왜 `FetchType.LAZY`로 선언되어 있는가? (즉시 로딩과의 차이, 지연 로딩이 실제로 풀리는 시점)

: 댓글 조회 시 불필요하게 member 정보까지 불러오지 않게 하기 위해 FetchType.LAZY가 선언되어 있습니다. 

- `comment.getMember().getNickname()`을 호출하는 순간 정확히 무슨 일이 일어나는가? (영속성 컨텍스트에 해당 Member가 없으면 SELECT 1건이 추가로 나가는 과정을 구체적으로 그려본다)

: 해당 Member를 조회하는 select 쿼리가 발생한다. 

- 댓글이 N건이면 왜 쿼리가 N+1개 나가는가? ("1"은 어디서 나온 쿼리이고 "N"은 어디서 나온 쿼리인가)

: 1은 comment를 조회하는 쿼리이고, 그 댓글 수가 N이면 그 N 번만큼 member를 개별로 조회하는 쿼리가 발생하므로 N + 1이라고 한다. 

- `StudyQueryService`의 클래스 주석은 `open-in-view: false`와 `@Transactional(readOnly = true)`를 왜 언급하는가? 컨트롤러에서 바로 지연 로딩을 건드리면 어떤 예외가 나는가?

1. `open-in-view: false`를 하면 데이터베이스 커넥션과 영속성 컨텍스트의 유지 범위가 웹 요청 전체가 아닌 트랜잭션 범위로 축소된다. @Transactional(readOnly = true) 가 있는 메서드 안에서만 영속성 컨텍스트가 유지된다.
2. 컨트롤러에서 지연 로딩된 연관 객체를 조회하면 LazyInitializationException이 발생한다.

## 2. 쿼리 수를 세는 장치

- 현재 프로젝트에는 실행 쿼리 수를 자동으로 세는 장치가 있는가, 없는가? (`application.yml`의 `org.hibernate.SQL: debug` 로그와 "쿼리 수를 세는 장치"는 왜 다른가 — 로그는 사람이 눈으로 세야 하고, 장치는 숫자로 바로 확인 가능해야 한다)

: 없지만 SqlCaptureInspector, JpaObservation 클래스를 구현해서 사용하여 쿼리 수를 셀 수 있다. 

```
public final class JpaObservation {
    private final Statistics statistics;

    public JpaObservation(EntityManagerFactory entityManagerFactory) {
        this.statistics = entityManagerFactory.unwrap(SessionFactory.class).getStatistics();
    }

    public void reset() {
        statistics.clear();
        SqlCaptureInspector.clear();
    }

    public long preparedStatementCount() {
        return statistics.getPrepareStatementCount();
    }

    public List<String> statements() {
        return SqlCaptureInspector.statements();
    }
}

public final class SqlCaptureInspector implements StatementInspector {
    private static final List<String> STATEMENTS = new CopyOnWriteArrayList<>();

    @Override
    public String inspect(String sql) {
        if (sql != null && !sql.isBlank()) {
            STATEMENTS.add(sql.replaceAll("\\s+", " ").trim());
        }
        return sql;
    }

    static void clear() {
        STATEMENTS.clear();
    }

    static List<String> statements() {
        return List.copyOf(STATEMENTS);
    }
}

spring.jpa.properties.hibernate.generate_statistics=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.session_factory.statement_inspector=co.dingcodingco.challenge.SqlCaptureInspector
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.orm.jdbc.bind=TRACE

```

- 쿼리 수를 세는 방법 후보에는 무엇이 있는가? (Hibernate Statistics `hibernate.generate_statistics` + `SessionFactory.getStatistics().getPrepareStatementCount()`, datasource-proxy/p6spy 같은 JDBC 프록시, 테스트에서 직접 카운트하는 커스텀 `Interceptor` 등)

: hibernate.generate_statistics` + `SessionFactory.getStatistics().getPrepareStatementCount()

혹시나 다른 방법이 있다면 구현 가이드에서 제공 받아서 구현하고자 한다. 

- 이번 미션에서는 프로덕션 코드에 상시 장치를 남길 것인가, 테스트/로컬 전용으로만 붙일 것인가? 그 판단 기준은 무엇인가?

: 테스트/로컬 전용으로만 붙인다. `JpaObservation`은 Hibernate Statistics를 감싼 것이고 `SqlCaptureInspector`는 모든 SQL을 메모리 리스트에 계속 쌓는 구조라, 프로덕션 트래픽에서 상시 켜두면 메모리 누수·오버헤드가 생긴다. 판단 기준은 "이 장치가 없으면 문제를 재현/검증할 수 없는가"이다. 쿼리 수 측정은 개선 전후를 비교하는 개발/테스트 단계에서만 필요하고, 운영 중 상시 모니터링이 필요하면 별도의 경량 지표(APM, 로그 기반 샘플링)로 가져가야 한다.

## 3. 병목 후보 네 가지의 차이

- N+1은 무엇이 문제인가? (요청 하나당 쿼리 수가 데이터 건수에 비례해서 늘어남 — "여러 번의 왕복"이 비용)

: 쿼리 1회로 가져올 수 있는 데이터를, N + 1 회의 쿼리로 가져오고, N회가 발생하는 것을 인지하지 못할 수 있다. 

- 벌크 연산은 무엇을 가리키는가? (한 건씩 갱신/삭제하는 대신 `UPDATE ... WHERE ...` 한 방으로 여러 행을 한 번에 처리하는 것 — 현재 프로젝트에서 벌크 연산이 필요할 만한 지점이 있는가?)

: 쿼리 1개로 insert/update/delete 를 하는 것. 여기서는 벌크 연산할 만한 곳은 없다. 

- Stream 필터 오버헤드는 무엇을 가리키는가? (DB에서 이미 걸러올 수 있는 조건을 애플리케이션 메모리로 다 가져온 뒤 `Stream.filter()`로 거르는 경우 — 이번 프로젝트 어디에 해당할 수 있는가?)

: DB에서 이미 걸러올 수 있는 조건을 애플리케이션 메모리로 다 가져온 뒤 `Stream.filter()`로 거르는 경우인데, 여기서는 filter하는 경우는 없다. 

- 비동기 처리는 무엇을 가리키는가? (동기로 순차 실행되던 여러 독립적인 작업을 병렬로 처리해 전체 응답 시간을 줄이는 것 — 현재 `comments()`처럼 순수 조회 하나짜리 API에도 적용 여지가 있는가, 없는가?)

: 동기로 순차 실행되던 독립적인 작업을 병렬로 처리해 전체 응답 시간을 줄이는 것.
현재는 없다. 

- 네 가지 중 댓글 목록 API(`GET /api/studies/{id}/comments`)에 가장 먼저 적용해야 할 것은 무엇이고, 그 근거는 무엇인가? (미션 문서가 "N+1을 고르면 여기서 시작한다"고 이미 힌트를 준 이유는 무엇인가 — 지연 로딩 선언과 클래스 주석이 이미 N+1을 가리키고 있다)

: N + 1 이다. 지연로딩되 회원 정보를 가져오는데서 N + 1이 발생한다. 

## 4. N+1 해결 기법 세 가지 비교

- `@BatchSize`는 어떻게 N+1을 줄이는가? (댓글 N건의 서로 다른 member_id를 모아 `IN (?, ?, ...)` 쿼리 하나로 배치 조회 — 완전히 1개로 줄이는 것이 아니라 "N번"을 "N/배치크기번"으로 줄인다는 차이)

: 댓글 N 건의 서로 다른 member_id를 모아 IN (?, ?, ...) 쿼리 하나로 배치 조회

- fetch join은 어떻게 N+1을 없애는가? (`JOIN FETCH`로 댓글과 회원을 한 번의 SELECT로 같이 가져옴 — 완전히 1개 쿼리로 줄어드는 이유)


: fetch join을 하면 JPQL join fetch 쿼리로 한 번의 select.. inner join 쿼리로 댓글과 회원 데이터를 같이 가져옴

- `@EntityGraph`는 fetch join과 무엇이 다른가? (선언적으로 즉시 로딩 전략을 지정하는 방식 — 리포지토리 메서드 시그니처는 그대로 두고 어노테이션만 추가하는 방식의 장단점)

@EntityGraph: 어노테이션 기반, Left outer join, Spring Data JPA 메서드 시그니처 그대로 활용
fetch join : JPQL, Querydsl 쿼리문 직접 작성, Inner Join(기본값, 변경 가능), 쿼리마다 join fetch 문을 작성해야함 

- 세 기법 모두 댓글-회원 관계에서 쓸 수 있다면, 이번 프로젝트 상황(댓글 수·조회 패턴)에서 어느 것이 더 적합한가? 그 판단 근거는 무엇인가?

페이징이 있는 경우 : EntityGraph
하지만 페이징이 없는 프로젝트 상황이므로, fetch join 이 적합하다고 판단됨. 


- 지금 `CommentRepository.findByStudyIdOrderByIdAsc`는 `List<Comment>`를 반환한다. fetch join으로 바꾸면 반환 타입이나 쿼리 자체를 어떻게 바꿔야 하는가?

: 반환타입은 바꿀 필요가 없이 Comment 객체에 Member 데이터를 담은 채로 반환된다. 

## 5. 측정 조건 고정

- 개선 전 쿼리 수를 재현하려면 무엇을 고정해야 하는가? (같은 studyId, 같은 댓글 수 — 시드 기본값은 `commentsPerStudy: 30`)

: 같은 studyId, 같은 댓글 수를 고정

- 쿼리 수는 몇 건이 나올 것으로 예상하는가? (댓글 30건 기준 1(댓글 목록) + 30(회원 개별 조회) = 31건이라는 계산을 실제 로그/장치로 확인하기 전에 먼저 예측해본다)

: 31건 (댓글 조회 1 회 + 회원 조회 30건)

- 개선 후 쿼리 수는 몇 건으로 줄어들 것으로 예상하는가? (fetch join이면 1건, `@BatchSize(30)`이면 이론상 2건까지 줄 수 있음 — 실제 배치 크기와 댓글 수 관계를 따져본다)

: fetch join 1건

- 쿼리 수 외에 응답 시간도 같이 측정해야 하는 이유는 무엇인가? (쿼리 수가 줄어도 응답 시간이 그대로면 다른 병목이 있다는 신호 — 두 지표가 같은 방향으로 움직이는지 확인해야 근거가 된다)

: 쿼리 수가 줄어도 응답 시간이 그대로면 다른 병목이 있을 수 있다. 

## 6. 회귀 테스트

- 개선 전후 회귀 테스트는 무엇을 같아야 한다고 검증해야 하는가? (댓글 개수, 각 댓글의 id·content·author 값이 개선 전후 동일해야 한다 — N+1을 없애는 과정에서 응답 계약이 바뀌면 안 된다)

: 댓글 개수, 각 댓글의 데이터가 동일해야 함 

- 쿼리 수 자체를 회귀 테스트의 assert 대상으로 넣을 것인가? 그렇다면 어떤 장치로 테스트 안에서 쿼리 수를 세는가?

: 쿼리 수는 변동이 있을 수 있어 회귀테스트 대상으로 적절하지 못하다. 
hibernate.generate_statistics` + `SessionFactory.getStatistics().getPrepareStatementCount() 를 통해 쿼리 수를 가져 올 수 있다. 

## 7. 리뷰 반영 (1~4주차 review-notes.md 체크리스트 적용)

- 4주차 리뷰에서 지적된 "개선 전 재현 테스트가 assert 없이 출력만 한다"는 패턴을 5주차에서는 어떻게 피할 것인가? (쿼리 수 측정 장치 자체를 assert 가능한 형태로 설계하면, 개선 전/후 모두 자동 검증이 가능해진다)

: 쿼리 수 자체는 6번에서 정리했듯 변동 가능성이 있어 정확한 숫자로 assert하지 않는다. 대신 "개선 전 쿼리 수 > 개선 후 쿼리 수" 같은 상대적 비교나, 개선 후 쿼리 수가 특정 상한(예: 1~2건) 이하인지를 assert 대상으로 삼는다. 즉 절대값 출력에 의존하지 않고, `preparedStatementCount()` 값을 테스트 코드 안에서 직접 읽어 비교/상한 검증에 사용한다. 응답 데이터(댓글 개수·id·content·author)는 개선 전후 완전히 동일해야 하므로 이 부분은 정확한 값으로 assert한다.

## 8. 근거형 질문 준비 (미션 질문 1~4)

정답은 안 적는다. 각 질문에서 무엇을 실측/조사해야 답이 나오는지만 짚는다.

- N+1 / 벌크 연산 / 스트림 FilterOverhead / 비동기 처리 중, 어떤 항목을 적용했고 왜 그 부분을 선택했나요?
  - 확인할 것: `Comment.member`의 지연 로딩 선언과 `StudyQueryService.comments()`의 주석, 실제 쿼리 수 측정 결과

- 구체적으로 어떤 방식으로 개선(또는 적용)했나요?
  - 확인할 것: `@BatchSize`·fetch join·`@EntityGraph` 중 선택한 근거, `CommentRepository`/`StudyQueryService` 변경 내용

- 모니터링(또는 로그) 결과, 개선 전후 무엇이 얼마나 좋아졌나요?
  - 확인할 것: 개선 전/후 쿼리 수, 응답 시간, 같은 studyId·같은 댓글 수 조건 고정 여부

- 개선 과정에서 겪은 문제나 고려해야 할 사항은 무엇이었나요?
  - 확인할 것: 회귀 테스트로 응답 계약이 동일한지 확인한 과정, 쿼리 수 측정 장치를 프로덕션에 남길지 여부의 판단
