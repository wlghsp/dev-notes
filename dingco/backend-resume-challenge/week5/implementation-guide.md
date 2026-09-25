# 5주차 구현 가이드

`week5/prep-questions.md`의 답을 전제로, `missions/README.md` Week 5 요구사항과 `studypass`(`/Users/jihochoi/Documents/study/dingco/challenge-backend-resume-2026-08-wlghsp-r17`) 현재 코드를 근거로 "무엇을 어떤 순서로 구현할지"만 정리한다. 실제 코드는 지호님이 직접 작성한다. 실행 결과·수치·판단 근거는 실제로 실행한 뒤 evidence와 prep-questions에 채운다.

---

## 0. 현재 코드 상태 확인

N+1 지점은 이미 주석으로 표시되어 있다.

- `Comment.member` (`Comment.java`): `FetchType.LAZY`로 선언되어 있고, "이 N+1을 관측하고 없애는 것이 5주차 미션이다"라고 명시.
- `CommentRepository.findByStudyIdOrderByIdAsc` (`CommentRepository.java`): 작성자를 함께 가져오지 않는다는 주석.
- `StudyQueryService.comments()` (`StudyQueryService.java`): `@Transactional(readOnly = true)` 안에서 `comment.getMember().getNickname()`을 호출해 지연 로딩을 실제로 터뜨린다. 클래스 주석에 `open-in-view: false`와의 관계도 이미 적혀 있다.
- `StudyController.comments()` (`StudyController.java`): `GET /api/studies/{studyId}/comments`로 이미 노출되어 있다. 컨트롤러 변경은 필요 없다.
- `application.yml`의 `app.seed.commentsPerStudy: 30` — 로컬 실행 시 스터디마다 댓글 30건이 시드된다.

## 1단계: 브랜치 확인

`submit/week-05__weekly-pr` 브랜치가 이미 생성되어 있고 원격에도 푸시되어 있다. 아직 main과 diff가 없는 상태(작업 시작 전)이므로 이 브랜치에 이어서 작업한다.

- [x] 브랜치 존재 확인 (`submit/week-05__weekly-pr`)

## 2단계: 쿼리 수를 세는 장치 붙이기

미션 1번(필수)이자 6번 근거형 질문("모니터링 결과")의 전제 조건이다. prep-questions 2번에서 정리한 대로 후보는 Hibernate Statistics 방식이다.

`application.yml`에 추가:

```yaml
spring:
  jpa:
    properties:
      hibernate:
        generate_statistics: true
```

측정용 컴포넌트를 새로 만든다. prep-questions 2번 답변대로 **테스트/로컬 전용**으로 두되, 지금은 프로덕션 코드(`src/main/`)에 두고 실제로 켜져 있을 때만 값을 읽는 방식으로 구현하는 것이 미션 요구사항("src/main/에서 개선")과도 맞는다 — 장치 자체는 `src/main/`에 두고, "상시 노출할 엔드포인트는 만들지 않는다"는 선을 지키는 것으로 판단 기준을 좁힌다.

```java
package co.dingcodingco.studypass.support;

import org.hibernate.SessionFactory;
import org.hibernate.stat.Statistics;
import org.springframework.stereotype.Component;
import jakarta.persistence.EntityManagerFactory;

@Component
public class QueryCountProbe {
    private final Statistics statistics;

    public QueryCountProbe(EntityManagerFactory entityManagerFactory) {
        this.statistics = entityManagerFactory.unwrap(SessionFactory.class).getStatistics();
    }

    public void reset() {
        statistics.clear();
    }

    public long queryCount() {
        return statistics.getPrepareStatementCount();
    }
}
```

측정은 테스트에서 `@Autowired QueryCountProbe`로 `reset()` 후 API 호출, 그다음 `queryCount()`를 읽는 흐름으로 한다. 별도로 SQL 문장 자체를 보고 싶으면 `logging.level.org.hibernate.SQL: debug`(이미 설정돼 있음)로 로그를 눈으로 확인한다.

**어느 `application.yml`에 넣을지**

- 로컬 실행(`src/main/resources/application.yml`)과 테스트(`src/test/resources/application.yml`) 둘 다에 넣는다: 로컬 실행이 3·5단계 실측의 대상이고, 테스트는 6단계 회귀 테스트가 `queryCountProbe.queryCount()`를 assert하는 대상이기 때문이다.
- prep-questions 2번의 "테스트/로컬 전용"이라는 판단은 운영 배포 프로파일(있다면)에는 넣지 않는다는 뜻으로 좁혀 적용한다.

- [ ] `hibernate.generate_statistics: true` 설정 추가
- [ ] `QueryCountProbe` 컴포넌트 추가
- [ ] 로컬에서 `GET /api/studies/{studyId}/comments` 호출 후 `queryCount()`로 개선 전 쿼리 수 확인

**실행 명령어**

애플리케이션을 로컬로 띄운 상태(`docker compose up -d`로 MySQL·Redis 기동 후 `./gradlew bootRun`)에서, studyId 하나를 정해 호출한다. 응답 시간은 curl의 `-w` 옵션으로 같이 뽑는다.

```bash
curl -s -o /dev/null -w "status=%{http_code} time=%{time_total}s\n" \
  "http://localhost:8080/api/studies/{studyId}/comments"
```

이 호출 시점에 애플리케이션 콘솔(`bootRun`을 띄운 터미널)에 `org.hibernate.SQL: debug` 로그로 실제 나간 SQL이 그대로 찍힌다. 그 콘솔 출력을 아래에 붙여넣는다.

### 실행 로그 (여기에 붙여넣기)

```
jihochoi@Jiho-MacBook-Pro challenge-backend-resume-2026-08-wlghsp-r17 % curl -s -o /dev/null -w "status=%{http_code} time=%{time_total}s\n" \
  "http://localhost:8080/api/studies/1/comments"
status=200 time=0.178303s

2026-09-25T12:26:54.964+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        c1_0.id,
        c1_0.content,
        c1_0.created_at,
        c1_0.member_id,
        c1_0.study_id 
    from
        study_comment c1_0 
    left join
        study s1_0 
            on s1_0.id=c1_0.study_id 
    where
        s1_0.id=? 
    order by
        c1_0.id
2026-09-25T12:26:54.966+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1]
2026-09-25T12:26:55.002+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.002+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1552]
2026-09-25T12:26:55.006+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.007+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [818]
2026-09-25T12:26:55.008+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.009+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1590]
2026-09-25T12:26:55.011+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.011+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1832]
2026-09-25T12:26:55.012+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.012+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [925]
2026-09-25T12:26:55.014+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.015+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [727]
2026-09-25T12:26:55.016+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.016+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [959]
2026-09-25T12:26:55.017+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.017+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [231]
2026-09-25T12:26:55.018+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.019+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1889]
2026-09-25T12:26:55.020+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.020+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1050]
2026-09-25T12:26:55.022+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.022+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1479]
2026-09-25T12:26:55.023+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.024+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1511]
2026-09-25T12:26:55.025+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.025+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1980]
2026-09-25T12:26:55.026+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.026+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1220]
2026-09-25T12:26:55.027+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.027+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [115]
2026-09-25T12:26:55.029+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.029+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [627]
2026-09-25T12:26:55.031+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.031+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [734]
2026-09-25T12:26:55.032+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.032+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1835]
2026-09-25T12:26:55.033+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.034+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [348]
2026-09-25T12:26:55.035+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.035+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1012]
2026-09-25T12:26:55.036+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.036+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [518]
2026-09-25T12:26:55.038+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.038+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1477]
2026-09-25T12:26:55.039+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.039+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [23]
2026-09-25T12:26:55.041+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.041+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [623]
2026-09-25T12:26:55.042+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.042+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [553]
2026-09-25T12:26:55.043+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.043+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [1851]
2026-09-25T12:26:55.044+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.044+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [786]
2026-09-25T12:26:55.045+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.045+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [7]
2026-09-25T12:26:55.047+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?
2026-09-25T12:26:55.047+09:00 TRACE 6611 --- [nio-8080-exec-4] org.hibernate.orm.jdbc.bind              : binding parameter (1:BIGINT) <- [539]
2026-09-25T12:26:55.048+09:00 DEBUG 6611 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname 
    from
        member m1_0 
    where
        m1_0.id=?

```

### 측정 기록 (evidence용 — 위 로그에서 추출, Claude가 채움)

```
studyId: 1
queryCount() 결과(SQL 로그에서 SELECT 문 개수를 센 값): 31건 (댓글 목록 1건 + 회원 단건 조회 30건)
응답 시간: 0.178303s (178ms)
```

로그에서 확인된 점: 댓글 목록 쿼리 1건(`study_comment ... left join study ...`) 뒤에 `member` 단건 조회(`where m1_0.id=?`)가 30번 반복된다. prep-questions 5번 예측(31건)과 정확히 일치한다. studyId=1은 댓글 30건 스터디로 확인됐으므로 이후 3·5·6단계에서도 studyId=1로 고정한다.

## 3단계: 병목이라는 근거를 먼저 남기기

미션 2번(필수)이다. prep-questions 3번에서 이미 N+1을 골랐고 근거(지연 로딩 선언 + 클래스 주석)도 정리했다. 여기서 새로 측정할 것은 없다 — **2단계에서 이미 나온 측정값을 "이게 병목이라는 근거"로 그대로 쓰는 단계**다.

- [x] 시드된 스터디 중 댓글이 30건인 studyId를 하나 고른다 → **studyId=1** (2단계에서 확인)
- [x] 개선 전 쿼리 수 확인 → **31건** (2단계 SQL 로그: 댓글 목록 조회 1건 + `member` 단건 조회 30번)
- [x] prep-questions 5번 예측(31건)과 일치하는지 확인 → **정확히 일치**
- [x] 응답 시간 확인 → **178ms** (2단계 curl 결과)

### 병목 근거 정리 (evidence용)

```
studyId: 1
댓글 수: 30
개선 전 쿼리 수: 31건 (댓글 목록 조회 1건 + 회원 단건 조회 30건)
개선 전 응답 시간: 178ms
근거: SQL 로그상 `study_comment` 조회 1건 이후 `member ... where m1_0.id=?`가
댓글 수(30)만큼 반복된다. 댓글 목록에서 comment.getMember().getNickname()을
호출하는 시점마다 Comment.member(FetchType.LAZY)가 개별로 풀리기 때문이다.
prep-questions 5번 예측(31건)과 정확히 일치해 N+1이 실제 병목임이 측정값으로 확인됐다.
```

## 4단계: N+1 개선 (fetch join)

미션 3번(필수)이다. prep-questions 4번에서 fetch join을 선택한 근거(페이징 없는 조회 패턴)를 이미 정리했다. `CommentRepository`에 fetch join 쿼리를 추가한다.

```java
package co.dingcodingco.studypass.comment;

import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface CommentRepository extends JpaRepository<Comment, Long> {

    List<Comment> findByStudyIdOrderByIdAsc(Long studyId);

    /**
     * 5주차 미션: fetch join으로 Comment와 Member를 한 번의 SELECT로 가져온다.
     * StudyQueryService.comments()가 이 메서드로 교체된다.
     */
    @Query("select c from Comment c join fetch c.member where c.study.id = :studyId order by c.id asc")
    List<Comment> findByStudyIdWithMemberOrderByIdAsc(@Param("studyId") Long studyId);
}
```

기존 `findByStudyIdOrderByIdAsc`는 지우지 않는다 — 개선 전 재현 테스트(6단계)가 여전히 이 메서드로 개선 전 상황을 재현해야 하기 때문이다(4주차 때는 기존 쿼리를 그 자리에서 교체했지만, 이번엔 미션이 "개선 전/후 같은 조건 재측정"을 요구하므로 두 경로를 모두 남겨야 비교가 가능하다).

`StudyQueryService.comments()`만 새 메서드를 쓰도록 교체한다.

```java
@Transactional(readOnly = true)
public List<Map<String, Object>> comments(Long studyId) {
    List<Comment> comments = commentRepository.findByStudyIdWithMemberOrderByIdAsc(studyId);
    return comments.stream()
            .map(comment -> Map.<String, Object>of(
                    "id", comment.getId(),
                    "content", comment.getContent(),
                    "author", comment.getMember().getNickname()))
            .toList();
}
```

- [ ] `CommentRepository`에 fetch join 메서드 추가(기존 메서드는 유지)
- [ ] `StudyQueryService.comments()`가 새 메서드를 쓰도록 교체
- [ ] 컨트롤러·응답 형식은 변경 없음 확인

코드는 studypass 저장소 파일에 그대로 남아 있으니 여기 다시 붙여넣지 않는다. evidence에는 파일 경로와 변경 요지만 짧게 적는다.

## 5단계: 개선 후 재측정

2단계와 **같은 studyId=1**로, 2단계와 같은 방식(curl + SQL 로그 눈으로 세기)으로 다시 측정한다.

- [ ] `GET /api/studies/1/comments` 호출. prep-questions 5번 예측(1건)과 일치하는지 확인.
- [ ] 응답 시간도 같은 방식으로 재측정.
- [ ] 개선 전/후 수치를 나란히 비교(쿼리 수: 31 → 1 등, 응답 시간 변화).

**실행 명령어**

fetch join 적용 후 애플리케이션을 재시작(`./gradlew bootRun`)하고 **같은 studyId=1**로 다시 호출한다.

```bash
curl -s -o /dev/null -w "status=%{http_code} time=%{time_total}s\n" \
  "http://localhost:8080/api/studies/1/comments"
```

### 실행 로그 (여기에 붙여넣기)

```
jihochoi@Jiho-MacBook-Pro challenge-backend-resume-2026-08-wlghsp-r17 % curl -s -o /dev/null -w "status=%{http_code} time=%{time_total}s\n" \
  "http://localhost:8080/api/studies/1/comments"
status=200 time=0.013656s

2026-09-25T12:56:00.193+09:00 DEBUG 16879 --- [nio-8080-exec-4] org.hibernate.SQL                        : 
    select
        c1_0.id,
        c1_0.content,
        c1_0.created_at,
        c1_0.member_id,
        m1_0.id,
        m1_0.created_at,
        m1_0.email,
        m1_0.nickname,
        c1_0.study_id 
    from
        study_comment c1_0 
    join
        member m1_0 
            on m1_0.id=c1_0.member_id 
    where
        c1_0.study_id=? 
    order by
        c1_0.id

```

### 측정 기록 (evidence용 — 개선 후)

```
studyId: 1 (2단계와 동일)
예측 쿼리 수: 1 (fetch join)
실제 쿼리 수: 1건 — SQL 로그에 select가 정확히 한 번만 찍혔고, study_comment와 member를
             join(fetch join)으로 한 번에 가져온다. 예측과 정확히 일치.
응답 시간: 0.013656s (13.7ms)

개선 전/후 비교:
  쿼리 수:   31건 → 1건   (약 31배 감소)
  응답 시간: 178ms → 13.7ms (약 13배 감소)
```

## 6단계: 회귀 테스트

미션 4번(필수)이자 prep-questions 6·7번에서 정리한 설계를 그대로 적용한다. 기존 `StudyControllerTest` 스타일(`@SpringBootTest(webEnvironment = RANDOM_PORT)` + `TestRestTemplate`)을 따른다. 새 파일 `CommentQueryTest.java`로 분리한다.

검증 대상은 두 가지다.

1. **응답 계약 동일성** — 개선 전/후 댓글 개수·id·content·author가 같아야 한다. `StudyQueryService`는 이제 fetch join 버전만 쓰므로, "개선 전"을 실제로 재현하려면 `commentRepository.findByStudyIdOrderByIdAsc(studyId)`를 테스트에서 직접 호출해 얻은 결과와, API가 돌려주는 결과를 같은 studyId로 비교한다.
2. **쿼리 수가 크게 줄었는지** — prep-questions 7번 판단대로 절대값(`=1`)으로 고정하지 않는다. `@BeforeEach`로 댓글 N건을 저장해두고, "개선 후 쿼리 수 < 댓글 수"(N+1이었다면 최소 N+1이 나왔을 자리)로 상한을 assert한다. 이렇게 하면 fetch join 내부 쿼리 형태가 바뀌어도(예: 2건으로 늘어나도) 테스트가 깨지지 않으면서, N+1이 사라졌다는 사실은 여전히 검증된다.

테스트 DB(H2)에는 시드가 꺼져 있으므로(`app.seed.enabled: false`), `@BeforeEach`에서 study·member·comment를 직접 저장해 studyId를 확보한다. 아래는 그 구조를 보여주는 스켈레톤이다.

```java
package co.dingcodingco.studypass.study;

import co.dingcodingco.studypass.comment.Comment;
import co.dingcodingco.studypass.comment.CommentRepository;
import co.dingcodingco.studypass.member.Member;
import co.dingcodingco.studypass.member.MemberRepository;
import co.dingcodingco.studypass.support.QueryCountProbe;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;
import java.util.UUID;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CommentQueryTest {

    private static final int COMMENT_COUNT = 5;

    @Autowired
    private TestRestTemplate restTemplate;
    @Autowired
    private StudyRepository studyRepository;
    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private CommentRepository commentRepository;
    @Autowired
    private QueryCountProbe queryCountProbe;

    private Long studyId;

    @BeforeEach
    void setUp() {
        Study study = studyRepository.save(
                new Study("댓글 테스트 스터디", "BACKEND", 10000, 5, LocalDateTime.now()));
        studyId = study.getId();

        for (int i = 0; i < COMMENT_COUNT; i++) {
            // 테스트 간 데이터가 롤백되지 않고 그대로 쌓이므로(EnrollmentConcurrencyTest와 동일한 이유),
            // email은 매번 유일한 값으로 만든다. 고정 문자열이면 두 번째 테스트의 setUp()에서
            // unique 제약 위반(ConstraintViolationException)이 난다.
            String suffix = UUID.randomUUID().toString();
            Member member = memberRepository.save(
                    new Member("comment-test-" + suffix + "@studypass.test", "댓글테스트회원" + i));
            commentRepository.save(new Comment(study, member, "댓글 내용 " + i));
        }
    }

    @DisplayName("fetch join 적용 후에도 댓글 목록 응답 계약(개수·id·content·author)은 동일하다")
    @Test
    void commentsResponseMatchesLazyLoadedResult() {
        // findByStudyIdWithMemberOrderByIdAsc(fetch join 버전)로 가져와야 트랜잭션 밖에서도
        // comment.getMember()를 안전하게 읽을 수 있다. @Transactional로 이 메서드를 감싸면
        // restTemplate.getForEntity()가 보내는 실제 HTTP 요청이 별도 커넥션에서 처리되면서
        // 아직 커밋되지 않은 setUp() 데이터를 못 보는 문제가 생기므로 트랜잭션은 걸지 않는다.
        List<Comment> expected = commentRepository.findByStudyIdWithMemberOrderByIdAsc(studyId);

        ResponseEntity<List> response = restTemplate.getForEntity(
                "/api/studies/" + studyId + "/comments", List.class);

        List<Map<String, Object>> actual = response.getBody();

        assertThat(actual).hasSize(expected.size());
        for (int i = 0; i < expected.size(); i++) {
            assertThat(actual.get(i).get("id")).isEqualTo(expected.get(i).getId().intValue());
            assertThat(actual.get(i).get("content")).isEqualTo(expected.get(i).getContent());
            assertThat(actual.get(i).get("author")).isEqualTo(expected.get(i).getMember().getNickname());
        }
    }

    @DisplayName("fetch join 적용 후 댓글 목록 조회 쿼리 수는 댓글 수보다 적다(N+1이 사라졌다)")
    @Test
    void commentsQueryCountIsLessThanCommentCount() {
        queryCountProbe.reset();
        restTemplate.getForEntity("/api/studies/" + studyId + "/comments", List.class);

        // N+1이었다면 최소 COMMENT_COUNT + 1건이 나왔을 것이다.
        // fetch join 적용 후에는 그보다 훨씬 적어야 한다(이론상 1건).
        assertThat(queryCountProbe.queryCount()).isLessThan(COMMENT_COUNT);
    }
}
```

`ResponseEntity<List>`로 받으면 각 원소는 `LinkedHashMap`으로 역직렬화되므로 `Map<String, Object>`로 다루는 데 문제없다. 위 코드는 구조를 보여주는 스켈레톤이므로, 실제 컴파일·실행하며 세부(패키지 경로, import 등)는 직접 맞춘다.

- [x] 개선 전/후 응답 계약 동일성 테스트
- [x] 개선 후 쿼리 수 assert 테스트
- [x] `./gradlew test --tests '*CommentQueryTest*'` 통과 확인

여기서 쓰는 studyId는 `@BeforeEach`가 테스트 DB(H2)에 새로 만든 study의 id이며, 2·5단계 로컬 측정의 studyId=1과는 다른 값이다.

**실행 명령어**

```bash
./gradlew test --tests '*CommentQueryTest*'
```

### 실행 로그

```
> Task :test

[Incubating] Problems report is available at: file:///.../build/reports/problems/problems-report.html

BUILD SUCCESSFUL in 9s
5 actionable tasks: 2 executed, 3 up-to-date
```

### 실행 기록 (evidence용)

```
실행 커맨드: ./gradlew test --tests '*CommentQueryTest*'
결과: BUILD SUCCESSFUL, 2개 테스트 모두 통과
  - commentsResponseMatchesLazyLoadedResult: 응답 계약(개수·id·content·author) 동일성 확인
  - commentsQueryCountIsLessThanCommentCount: 개선 후 쿼리 수 < 댓글 수(5), N+1 해소 확인

시행착오 (evidence "겪은 문제" 항목에 활용):
1. 응답 계약 테스트에 @Transactional을 걸었더니 actual이 빈 배열로 나와 실패했다.
   원인: @Transactional이 걸린 테스트 메서드 안에서 TestRestTemplate으로 보낸 실제 HTTP 요청은
   별도 서블릿 스레드·별도 DB 커넥션에서 처리되는데, @BeforeEach가 저장한 데이터가 아직
   테스트 트랜잭션 안에서만 존재하고(커밋 전) 있어 서버 쪽에서 보이지 않았다.
   해결: @Transactional을 제거하고, expected 조회를 fetch join 메서드로 바꿔
   트랜잭션 없이도 comment.getMember()를 안전하게 읽도록 했다.
2. 그 다음 두 번째 테스트에서 ConstraintViolationException(email unique 제약 위반)이 났다.
   원인: @BeforeEach가 고정 문자열 이메일("comment-test-0@studypass.test" 등)을 쓰는데,
   테스트 간 데이터가 롤백되지 않고 누적되어 두 번째 테스트의 setUp()에서 같은 이메일이
   중복 insert됐다.
   해결: UUID.randomUUID()로 이메일을 매번 유일하게 생성(EnrollmentConcurrencyTest와 동일한 패턴).
```

## 7단계: resume·evidence·PR

`resume/resume.md` 문장 예시:

```text
댓글 목록 API에서 작성자 조회로 인한 N+1(댓글 [N]건 기준 쿼리 [N+1]건)을
Hibernate Statistics로 측정해 확인한 뒤, fetch join으로 쿼리 수를 1건으로 줄이고
응답 시간을 [개선 전]ms → [개선 후]ms로 개선했다.
```

`evidence/week-05__weekly-pr.md`에는 다음을 빠짐없이 남긴다.

- [ ] 쿼리 수 측정 장치(`QueryCountProbe` + `generate_statistics`)와 개선 전 쿼리 수·응답 시간
- [ ] N+1이 병목이라는 근거(이 가이드 3단계 "병목 근거 정리", prep-questions "3. 병목 후보 네 가지의 차이" 판단)
- [ ] fetch join을 선택한 이유(prep-questions "4. N+1 해결 기법 세 가지 비교", `@BatchSize`·`@EntityGraph`와 비교)
- [ ] 개선 후 같은 조건 재측정 결과(쿼리 수 1건, 응답 시간)
- [ ] 회귀 테스트 통과 출력(응답 계약 동일성 + 쿼리 수)
- [ ] 근거형 질문 1~4 답변
- [ ] `## 리뷰 반영`: 최초에는 `자동 리뷰 수신 전`, 이후 지적 유무에 따라 갱신

제출 계약 확인:

- [ ] `resume/resume.md`, `src/main/`, `src/test/`, `evidence/week-05__weekly-pr.md`를 모두 변경했다.
- [ ] `missions/`, `challenge.json`, 채점 workflow는 변경하지 않았다.
- [ ] 개선 전/후 쿼리 수·응답 시간이 저장소 안에 실제로 기록되어 있다.

---

## 8단계 (선택 확장): Grafana 대시보드로 지표 시각화

3단계 판단대로 벌크·스트림 필터·비동기는 이 프로젝트에 적용 여지가 없으므로, 선택 확장은 **Grafana 대시보드**로 간다. `docker-compose.yml`에 Prometheus·Grafana가 이미 2주차부터 떠 있고, `build.gradle`에도 `micrometer-registry-prometheus`가 이미 있으므로 인프라는 그대로 쓴다.

### 8-0. `hibernate.*` 메트릭이 노출되지 않는 문제 (실제로 겪은 트러블슈팅)

`hibernate.generate_statistics: true`와 `micrometer-registry-prometheus`만으로는 `/actuator/prometheus`에 `hibernate_*` 메트릭이 뜨지 않았다. 원인 두 가지를 순서대로 고쳤다.

1. **`session_factory.name`이 없었다** — Micrometer가 `EntityManagerFactory`를 등록하려면 이름표가 필요하다. `application.yml`에 추가:
   ```yaml
   spring:
     jpa:
       properties:
         hibernate:
           session_factory:
             name: studypass
   ```
2. **`org.hibernate.stat.HibernateMetrics` 클래스 자체가 클래스패스에 없었다** — `--debug`로 띄워 조건 평가 리포트를 보면 `HibernateMetricsAutoConfiguration`이 `@ConditionalOnClass`에서 막혀 조용히 스킵되고 있었다. Spring Boot 3 + Hibernate 6에서는 이 클래스가 `hibernate-micrometer`라는 별도 아티팩트에 있다. `build.gradle`에 추가:
   ```groovy
   runtimeOnly 'org.hibernate.orm:hibernate-micrometer'
   ```

둘 다 추가하고 재시작하니 `hibernate_statements_total{status="prepared"}` 등이 정상 노출됐다.

### 8-1. 메트릭이 실제로 노출되는지 확인

애플리케이션이 떠 있는 상태(`./gradlew bootRun`)에서 확인한다.

```bash
curl -s "http://localhost:8080/actuator/prometheus" | grep hibernate
```

`hibernate_statements_total{...,status="prepared"}`이 이 미션에서 쓸 지표다. `QueryCountProbe.queryCount()`(Hibernate `Statistics.getPrepareStatementCount()`)와 같은 값을 센다. 실제로 `GET /api/studies/1/comments` 호출 전후로 이 값이 1씩 오르는 것을 확인했다(fetch join 적용 후 기준).

응답 시간은 Spring MVC가 기본으로 노출하는 `http_server_requests_seconds`를 쓴다. `uri="/api/studies/{studyId}/comments"` 태그로 필터링한다.

```bash
curl -s "http://localhost:8080/actuator/prometheus" | grep 'http_server_requests_seconds.*comments'
```

### 8-2. Prometheus·Grafana 기동 및 확인

```bash
docker compose up -d prometheus grafana
```

- Prometheus(`http://localhost:9090`) → Status > Targets에서 `studypass` job이 `UP`인지 확인.
- Grafana(`http://localhost:3000`, admin/studypass, `monitoring/prometheus.yml` 기준 익명 접속도 허용) → Data source에 Prometheus가 이미 잡혀 있는지 확인.

### 8-3. 대시보드 패널 구성

새 대시보드를 만들고 패널 2개를 추가한다.

1. **쿼리 수 (개선 전/후 비교용 카운터)** — `hibernate_statements_total{status="prepared"}`을 그래프로. `rate()`가 아니라 누적 카운터 자체를 보는 게 이번 목적(요청 1건당 몇 건 나가는지)과 더 맞으므로, 패널 설명에 "댓글 목록 API 호출 시점 전후 값 차이로 읽는다"는 것을 적어둔다.
2. **댓글 목록 API 응답 시간** — `http_server_requests_seconds_sum{uri="/api/studies/{studyId}/comments"} / http_server_requests_seconds_count{uri="/api/studies/{studyId}/comments"}`로 평균 응답 시간 그래프.

### 8-4. 개선 전/후 캡처

fetch join 적용 전(4단계 이전 커밋으로 잠깐 되돌리거나, git stash로 4단계 변경분을 잠시 걷어낸 상태)과 적용 후 각각 `GET /api/studies/1/comments`를 호출해, 두 패널의 값 변화를 스크린샷으로 남긴다. 2·5단계에서 이미 curl+SQL 로그로 확보한 수치(31건→1건, 178ms→13.7ms)와 대시보드 그래프가 같은 방향으로 움직이는지 evidence에서 대조한다.

- [x] `hibernate` 관련 메트릭이 `/actuator/prometheus`에 실제로 노출되는지 확인
- [x] Grafana 대시보드에 쿼리 수·응답 시간 패널 추가
- [x] 개선 전/후 캡처 확보, 2·5단계 수치와 대조

### 메트릭 확인 기록 (evidence용)

```
build.gradle: runtimeOnly 'org.hibernate.orm:hibernate-micrometer' 추가
application.yml: spring.jpa.properties.hibernate.session_factory.name: studypass 추가

실제 노출된 메트릭: hibernate_statements_total{application="studypass",entityManagerFactory="entityManagerFactory",status="prepared"}
검증: GET /api/studies/1/comments(fetch join 적용 후) 호출 전 0.0 → 호출 후 1.0으로 정확히 1 증가.
     이 값이 QueryCountProbe.queryCount()(Statistics.getPrepareStatementCount())와 같은 지표라는 것을 확인.

Grafana 대시보드 캡처(개선 전/후 비교):
- 개선 후(fetch join) 상태에서 GET /api/studies/1/comments를 4회 호출 → 쿼리 수 카운터가
  7 → 8 → ... → 11로, 호출당 1건씩 작게 증가하는 계단이 그래프에 찍힘.
- 코드를 findByStudyIdOrderByIdAsc(N+1 버전)로 잠깐 되돌려 같은 API를 3회 호출 →
  0 → 93으로, 호출당 31건씩(댓글 1 + 회원 30) 크게 뛰는 계단이 같은 그래프에 이어서 찍힘.
- 같은 패널 안에서 "작은 계단(개선 후)"과 "훨씬 큰 계단(개선 전)"이 나란히 비교되어,
  2·5단계에서 curl+SQL 로그로 확인한 수치(31건→1건)와 같은 방향의 변화임을 대시보드로도 확인.
- 응답 시간 패널도 개선 전 호출 구간에서 평균 0.16s(160ms)로, 2단계 curl 측정값(178ms)과
  비슷한 수준으로 나타남. 캡처 후 코드는 다시 fetch join으로 원상복구하고 재확인(1건 증가)함.
```
