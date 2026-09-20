# Week 5 구현 가이드

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
prep-questions.md에서 확정한 내용대로 실제 코드를 작성하는 단계.

prep-questions.md에서 확정한 것:
- 동적 조회 조건: Todo 제목 부분일치(title Like) + 담당자(User username) 일치
- N+1 개선 방식: DTO projection
- 선택 과제(컬렉션 fetch join+페이징, 벌크 연산 정합성)는 이번 주차 제외 — 필수만

Week4 리뷰에서 지적됐던 `backCode` 오타와 `PaymentJoined` getter 누락은 지호님이 이미 직접 반영 완료(`BankTransferPaymentJoined`는 `bankCode`, `PaymentJoined`는 getter 4개 모두 존재) — 이번 주차에서는 추가 작업 없음.

기존 프로젝트 컨벤션을 그대로 따른다:
- `Todo`는 `co.dingcodingco.challenge.domain.todo` 패키지, `User`는 `co.dingcodingco.challenge.domain` 패키지(`TodoAssign`/`TodoIdentity`는 week2 실험용 단일 클래스라 루트 패키지에 그대로 둠)
- `Todo.user`는 이미 `@ManyToOne(fetch = FetchType.LAZY)`로 선언되어 있고 `User.todos`는 `mappedBy`
- `@DataJpaTest` + `JpaObservation`(`SqlCaptureInspector` 기반) 재사용
- `JPAQueryFactory`는 `QuerydslConfiguration`에 이미 Bean으로 등록되어 있음 — `@DataJpaTest`에서 쓰려면 해당 설정 클래스를 `@Import`해야 한다
- Q타입(`QTodo`, `QUser`)은 이미 생성되어 있음(`target/generated-sources`, `QTodo`는 `co.dingcodingco.challenge.domain.todo` 패키지)

---

## 1단계. 조건 DTO

패키지: `co.dingcodingco.challenge.domain.todo`(Todo와 같은 위치)

```java
package co.dingcodingco.challenge.domain.todo;

public record TodoSearchCondition(String title, String username) {
}
```

두 필드 모두 null을 허용한다 — 클라이언트가 조건을 하나만 넘기거나 아예 안 넘길 수 있어야 하기 때문이다(prep-questions A-2에서 이미 정리한 이유). 기존 엔티티(`Todo`, `User`, `PaymentJoined` 등)는 JPA 스펙 제약(기본 생성자, setter 없는 불변 필드 불가) 때문에 record를 쓸 수 없지만, `TodoSearchCondition`은 엔티티가 아닌 순수 DTO라 record가 적합하다 — 레포에 record 사용 전례는 없지만 Java 17이라 문제없이 쓸 수 있다. 접근자는 `getTitle()`이 아니라 `title()`이라는 점에 주의(record 컴포넌트 접근자 규칙).

---

## 2단계. DTO projection 결과 DTO

Todo + User를 조인해서 필요한 필드만 담는 결과 DTO. `@QueryProjection`을 쓰면 컴파일 타임에 생성자 시그니처 불일치를 잡을 수 있어 이 방식으로 간다(prep-questions A-3에서 정리한 트레이드오프 중 안전성 우선 선택).

```java
package co.dingcodingco.challenge.domain.todo;

import com.querydsl.core.annotations.QueryProjection;

public record TodoSearchResult(Long todoId, String title, String username) {

    @QueryProjection
    public TodoSearchResult {
    }
}
```

`@QueryProjection`을 record의 canonical(compact) constructor에 붙이는 방식이다. QueryDSL 5.1.0은 record를 지원하므로 이 형태로 `QTodoSearchResult`가 생성된다. `mvn compile`(또는 `mvn generate-sources`)을 한 번 실행해서 `QTodoSearchResult`가 `target/generated-sources`에 생성되는지 확인해야 한다. 기존 `querydsl-apt` 설정이 `co.dingcodingco.challenge` 패키지 전체를 스캔하므로 별도 설정 변경은 필요 없다. 접근자는 `getTitle()`이 아니라 `title()`, `getUsername()`이 아니라 `username()`이다.

---

## 3단계. Repository

패키지: `co.dingcodingco.challenge.domain.todo`

```java
package co.dingcodingco.challenge.domain.todo;

import com.querydsl.core.types.dsl.BooleanExpression;
import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.stereotype.Repository;
import org.springframework.util.StringUtils;

import java.util.List;

import static co.dingcodingco.challenge.domain.todo.QTodo.todo;
import static co.dingcodingco.challenge.domain.QUser.user;

@Repository
public class TodoQueryRepository {

    private final JPAQueryFactory queryFactory;

    public TodoQueryRepository(JPAQueryFactory queryFactory) {
        this.queryFactory = queryFactory;
    }

    public List<Todo> search(TodoSearchCondition condition) {
        return queryFactory
                .selectFrom(todo)
                .where(
                        titleContains(condition.title()),
                        usernameEq(condition.username())
                )
                .fetch();
    }

    public List<TodoSearchResult> searchWithProjection(TodoSearchCondition condition) {
        return queryFactory
                .select(new QTodoSearchResult(todo.id, todo.title, user.username))
                .from(todo)
                .join(todo.user, user)
                .where(
                        titleContains(condition.title()),
                        usernameEq(condition.username())
                )
                .fetch();
    }

    private BooleanExpression titleContains(String title) {
        return StringUtils.hasText(title) ? todo.title.contains(title) : null;
    }

    private BooleanExpression usernameEq(String username) {
        return StringUtils.hasText(username) ? todo.user.username.eq(username) : null;
    }
}
```

두 메서드를 나눈 이유: `search()`는 N+1을 그대로 재현하는 "개선 전" 버전(엔티티 조회, `user`는 LAZY라 접근 시 추가 쿼리 발생), `searchWithProjection()`은 join으로 한 번에 가져오는 "개선 후" 버전이다. 같은 `where` 조건 로직(`titleContains`, `usernameEq`)을 공유해서 조건 조합 자체는 "개선 전후" 비교에서 변수가 되지 않도록 했다.

`StringUtils.hasText()`는 null과 빈 문자열(`""`, `" "`) 모두를 걸러낸다 — prep-questions A-1에서 정리한 규칙 그대로다.

---

## 4단계. 테스트

패키지: `co.dingcodingco.challenge.week5`

### `TodoQueryRepositoryTest.java`

```java
package co.dingcodingco.challenge.week5;

import co.dingcodingco.challenge.JpaObservation;
import co.dingcodingco.challenge.QuerydslConfiguration;
import co.dingcodingco.challenge.domain.todo.Todo;
import co.dingcodingco.challenge.domain.todo.TodoQueryRepository;
import co.dingcodingco.challenge.domain.todo.TodoSearchCondition;
import co.dingcodingco.challenge.domain.todo.TodoSearchResult;
import co.dingcodingco.challenge.domain.User;
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.context.annotation.Import;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
@Import({QuerydslConfiguration.class, TodoQueryRepository.class})
class TodoQueryRepositoryTest {

    @Autowired
    EntityManager em;
    @Autowired
    EntityManagerFactory emf;
    @Autowired
    TodoQueryRepository todoQueryRepository;
    JpaObservation observation;

    @BeforeEach
    void setUp() {
        observation = new JpaObservation(emf);
    }

    private void createFixtures() {
        User user1 = new User("alice", "password123");
        User user2 = new User("bob", "password123");
        Todo todo1 = new Todo("보고서 작성");
        Todo todo2 = new Todo("보고서 검토");
        Todo todo3 = new Todo("회의 준비");
        todo1.setUser(user1);
        todo2.setUser(user2);
        todo3.setUser(user1);
        em.persist(user1);
        em.persist(user2);
        em.persist(todo1);
        em.persist(todo2);
        em.persist(todo3);
        em.flush();
        em.clear();
    }

    @DisplayName("제목 부분일치와 담당자 조건을 함께 걸면 두 조건을 모두 만족하는 Todo만 조회된다")
    @Test
    void search_combines_title_and_username_condition() {
        // given
        createFixtures();
        observation.reset();
        TodoSearchCondition condition = new TodoSearchCondition("보고서", "alice");

        // when
        List<Todo> result = todoQueryRepository.search(condition);

        // then
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getTitle()).isEqualTo("보고서 작성");
    }

    @DisplayName("조건이 null이면 해당 조건은 무시되고 나머지 조건만 적용된다")
    @Test
    void search_ignores_null_condition() {
        // given
        createFixtures();
        observation.reset();
        TodoSearchCondition condition = new TodoSearchCondition("보고서", null);

        // when
        List<Todo> result = todoQueryRepository.search(condition);

        // then
        assertThat(result).hasSize(2);
    }

    @DisplayName("개선 전(search)은 목록 조회 1번 + LAZY user 접근 시 N번, 총 1+N 쿼리가 발생한다")
    @Test
    void search_causes_n_plus_one_when_accessing_user() {
        // given
        createFixtures();
        observation.reset();
        TodoSearchCondition condition = new TodoSearchCondition("보고서", null);

        // when
        List<Todo> result = todoQueryRepository.search(condition);
        result.forEach(t -> t.getUser().getUsername());

        // then
        assertThat(result).hasSize(2);
        // 목록 조회 1 + 서로 다른 User 접근 2 (alice, bob) = 3
        assertThat(observation.statements()).hasSize(3);
    }

    @DisplayName("개선 후(searchWithProjection)는 join으로 한 번에 가져와 쿼리 1개만 발생한다")
    @Test
    void search_with_projection_avoids_n_plus_one() {
        // given
        createFixtures();
        observation.reset();
        TodoSearchCondition condition = new TodoSearchCondition("보고서", null);

        // when
        List<TodoSearchResult> result = todoQueryRepository.searchWithProjection(condition);

        // then
        assertThat(result).hasSize(2);
        assertThat(observation.statements()).hasSize(1);
    }

    @DisplayName("개선 전후 결과의 개수와 각 Todo의 title·username이 동일하다")
    @Test
    void search_and_projection_return_consistent_results() {
        // given
        createFixtures();
        TodoSearchCondition condition = new TodoSearchCondition("보고서", null);

        // when
        List<Todo> beforeResult = todoQueryRepository.search(condition);
        List<TodoSearchResult> afterResult = todoQueryRepository.searchWithProjection(condition);

        // then
        assertThat(afterResult).hasSize(beforeResult.size());
        List<String> beforeTitles = beforeResult.stream().map(Todo::getTitle).sorted().toList();
        List<String> afterTitles = afterResult.stream().map(TodoSearchResult::title).sorted().toList();
        assertThat(afterTitles).isEqualTo(beforeTitles);
    }
}
```

**주의**:
- `createFixtures()`에서 `em.flush()` + `em.clear()`로 1차 캐시를 비운 뒤, 각 테스트에서 `observation.reset()`을 그 다음에 호출한다 — prep-questions A-1에서 정리한 순서(fixture INSERT 이후 캐시 정리 → 측정 시작) 그대로다.
- `search_causes_n_plus_one_when_accessing_user`에서 쿼리 수를 3으로 고정한 근거: Todo 2건(보고서 작성-alice, 보고서 검토-bob)이 서로 다른 User를 참조하므로 목록 조회 1 + LAZY 초기화 2 = 3. Todo 수가 아니라 **서로 다른 User 수**가 추가 쿼리 수를 결정한다는 점에 주의 — 같은 User를 여러 Todo가 참조하면 영속성 컨텍스트 1차 캐시 덕분에 중복 조회가 안 나갈 수 있다. 이 실험 fixture는 의도적으로 alice에게 Todo 2개, bob에게 Todo 1개를 배정했지만 검색 조건("보고서")에 걸리는 건 alice의 것과 bob의 것 각 1개씩이라 두 User 모두 접근하게 된다.
- 이 숫자가 실제로 안 맞으면(Hibernate 버전, 캐시 여부에 따라 달라질 수 있음) 먼저 테스트를 돌려서 로그에 찍히는 실제 SQL과 개수를 확인하고, 그 다음에 assertion을 맞추면 된다 — week4에서도 같은 방식으로 갔다.

---

## 5단계. Repository/Controller 레이어 정리

prep-questions.md 6번에 "Repository/Controller 레이어가 아직 없어 이번 주차에 새로 만든다"고 되어 있는데, 미션 요구사항인 "API"까지 만들려면 `TodoQueryRepository` 위에 얇은 Controller 하나를 추가한다.

```java
package co.dingcodingco.challenge.domain.todo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class TodoQueryController {

    private final TodoQueryRepository todoQueryRepository;

    public TodoQueryController(TodoQueryRepository todoQueryRepository) {
        this.todoQueryRepository = todoQueryRepository;
    }

    @GetMapping("/todos")
    public List<TodoSearchResult> search(
            @RequestParam(required = false) String title,
            @RequestParam(required = false) String username
    ) {
        return todoQueryRepository.searchWithProjection(new TodoSearchCondition(title, username));
    }
}
```

`@DataJpaTest`만으로는 Controller까지 테스트하지 않는다(레포에 `@WebMvcTest`나 `@SpringBootTest` 사용 전례가 아직 없으므로, Controller는 컴파일 확인 수준으로만 두고 이번 주차 테스트는 Repository 레벨에 집중한다). Controller 계층 테스트가 필요하다고 판단되면 별도로 논의한다.

---

## 6단계. evidence 문서에 담을 것

`evidence/week-05__weekly-pr.md`(또는 프로젝트 규칙에 맞는 파일명)에 아래 내용을 채운다:

1. **변경**: `TodoSearchCondition`, `TodoSearchResult`, `TodoQueryRepository`, `TodoQueryController`, 테스트 목록
2. **검증**: 각 테스트 실행 결과 SQL 로그를 그대로 붙여넣기 — 특히 `search`(개선 전, 1+N개) vs `searchWithProjection`(개선 후, 1개) 쿼리 수 대비를 로그로 남긴다
3. **선택 근거**: prep-questions A-2/A-3에서 정리한 DTO projection 선택 이유(`@QueryProjection` vs `Projections.constructor` 판단 포함)
4. **근거형 질문 4개**: prep-questions.md A-1에서 정리한 답을 "최초 판단 → 연결한 코드/로그 → 검증 후 답변" 형식으로 재구성
5. **리뷰 반영**: Week1·Week3에서 반복 지적된 공백 문제 — "자동 리뷰 수신 전"만 적고 끝내지 않기(A-5 참고)

---

## 실행 순서 요약

1. `Todo`를 `co.dingcodingco.challenge.domain.todo` 패키지로 이동(완료 — `QTodo`도 같은 패키지로 재생성됨, `QuerydslGenerationTest`의 `QTodo` import만 수동 수정 필요했음)
2. `TodoSearchCondition`, `TodoSearchResult` 작성 → `mvn compile`로 `QTodoSearchResult` 생성 확인
3. `TodoQueryRepository` 작성(`search`, `searchWithProjection` 두 메서드)
4. `week5/TodoQueryRepositoryTest` 작성, 위 코드 그대로 작성
5. `mvn test`로 실행하고 로그에 찍히는 실제 SQL과 쿼리 수를 evidence에 옮겨 적기 — N+1 쿼리 수 assertion이 fixture와 안 맞으면 로그 보고 조정
6. `TodoQueryController` 작성(컴파일 확인 수준)
7. prep-questions.md 답변을 근거형 질문 형식으로 evidence에 재구성
8. PR 생성 전 `git status`/`gh pr diff`로 새 파일이 diff에 잡히는지 확인
