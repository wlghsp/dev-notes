# Week 3 — 계층 분리와 트랜잭션 (execution-guide)

prep-questions.md의 판단을 실제 구현 순서로 정리한 것. 대상 저장소는
`challenge-spring-boot-2026-09-wlghsp-r20`.

기존 코드 확인 결과 참고:
- `Todo` 클래스는 public 생성자(`title, completed`) + package-private 생성자(`id, title, completed`) + package-private `assignId()` 패턴을 쓴다. title 공백 체크를 생성자에서 한다. User도 같은 패턴으로 만든다.
- `JdbcTodoRepository.save()`는 현재 INSERT에 `user_id`를 안 넣고 있다. schema.sql엔 `todo.user_id` 컬럼이 이미 있으니, 이번 주 작업에서 여기를 고쳐야 한다.
- `TodoRepository`는 인터페이스 + `@Primary @Repository JdbcTodoRepository` 구현체 패턴. `UserRepository`도 동일하게 간다 (prep-questions 필수 1 판단대로).

## 기존 코드 확인 사항 (2026-09-26 시점)

`new Todo(title, completed)` 2-argument 생성자는 아래 6곳에서 이미 쓰이고 있다.
`TodoTest.java`(2곳), `TodoServiceTest.java`, `JdbcTodoRepositoryTest.java`,
`InMemoryTodoRepositoryTest.java`, `TodoService.create()`.

**결정: 기존 생성자는 그대로 두고, `userId`를 받는 생성자를 오버로드로 추가한다.**
기존 호출부/테스트는 손대지 않는다. `userId`가 필요한 조율 서비스에서만 새 생성자를 쓴다.

## 순서

### 1. User 도메인 + UserRepository

`src/main/java/co/dingcodingco/challenge/user/User.java` (새 패키지)

```java
package co.dingcodingco.challenge.user;

public class User {
    private Long id;
    private String name;

    public User(String name) {
        this(null, name);
    }

    User(Long id, String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or blank");
        }
        this.id = id;
        this.name = name;
    }

    void assignId(Long id) {
        this.id = id;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

`src/main/java/co/dingcodingco/challenge/user/UserRepository.java`

```java
package co.dingcodingco.challenge.user;

import java.util.Optional;

public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
    long count();
}
```

`count()`는 필수 2/3의 "User 1건/0건 확인" 검증에 쓸 카운트 메서드. `Todo`와의 대칭성보다
검증 요구사항이 우선이라 추가한다.

`src/main/java/co/dingcodingco/challenge/user/JdbcUserRepository.java`

```java
package co.dingcodingco.challenge.user;

import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;

import java.sql.PreparedStatement;
import java.util.List;
import java.util.Optional;

@Primary
@Repository
public class JdbcUserRepository implements UserRepository {
    private final JdbcTemplate jdbcTemplate;

    public JdbcUserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public User save(User user) {
        KeyHolder keyHolder = new GeneratedKeyHolder();

        jdbcTemplate.update(con -> {
            PreparedStatement ps = con.prepareStatement(
                    "insert into challenge_user (name) values (?)",
                    PreparedStatement.RETURN_GENERATED_KEYS);
            ps.setString(1, user.getName());
            return ps;
        }, keyHolder);
        user.assignId(keyHolder.getKey().longValue());
        return user;
    }

    @Override
    public Optional<User> findById(Long id) {
        List<User> result = jdbcTemplate.query(
                "select id, name from challenge_user where id = ?",
                (rs, rowNum) -> new User(rs.getLong("id"), rs.getString("name")),
                id);

        return result.stream().findFirst();
    }

    @Override
    public long count() {
        return jdbcTemplate.queryForObject("select count(*) from challenge_user", Long.class);
    }
}
```

기존 `JdbcTodoRepository`가 `private JdbcTemplate`(final 없음)로 선언돼 있는데,
새 코드는 `final`을 붙였다. 기존 코드 스타일을 그대로 따르고 싶으면 `final` 빼도 된다 — 취향 차이라 지호님 판단.

### 2. Todo에 user_id 연결

`Todo.java`에 `userId` 필드와 새 생성자 오버로드 추가 (기존 생성자는 유지):

```java
public class Todo {
    private Long id;
    private Long userId;
    private String title;
    private boolean completed;

    public Todo(String title, boolean completed) {
        this(null, null, title, completed);
    }

    public Todo(Long userId, String title, boolean completed) {
        this(null, userId, title, completed);
    }

    Todo(Long id, Long userId, String title, boolean completed) {
        if (title == null || title.isBlank()) {
            throw new IllegalArgumentException("Title cannot be null or blank");
        }
        this.id = id;
        this.userId = userId;
        this.title = title;
        this.completed = completed;
    }

    // 기존 assignId, getId, getTitle, isCompleted, complete 그대로
    // + getUserId() 추가

    public Long getUserId() {
        return userId;
    }
}
```

주의: package-private 생성자 `Todo(Long id, String title, boolean completed)`가
`Todo(Long id, Long userId, String title, boolean completed)`로 인자 순서/개수가 바뀐다.
이 생성자를 쓰는 곳은 `JdbcTodoRepository.findById()`의 람다뿐이므로 거기만 같이 고치면 된다
(위 6곳의 public 생성자 호출부와는 무관).

`JdbcTodoRepository.java` 수정:

```java
@Override
public Todo save(Todo todo) {
    KeyHolder keyHolder = new GeneratedKeyHolder();

    jdbcTemplate.update(con -> {
        PreparedStatement ps = con.prepareStatement(
                "insert into todo (user_id, title, completed) values (?, ?, ?)",
                PreparedStatement.RETURN_GENERATED_KEYS);
        ps.setObject(1, todo.getUserId());
        ps.setString(2, todo.getTitle());
        ps.setBoolean(3, todo.isCompleted());
        return ps;
    }, keyHolder);
    todo.assignId(keyHolder.getKey().longValue());
    return todo;
}

@Override
public Optional<Todo> findById(Long id) {
    List<Todo> result = jdbcTemplate.query(
            "select id, user_id, title, completed from todo where id = ?",
            (rs, rowNum) -> new Todo(rs.getLong("id"),
                    (Long) rs.getObject("user_id"),
                    rs.getString("title"),
                    rs.getBoolean("completed")),
            id);

    return result.stream().findFirst();
}
```

`ps.setObject(1, todo.getUserId())`를 쓴 이유: `userId`가 null일 수 있는 상황(기존 2-argument
생성자로 만든 Todo)이 남아있어서 `setLong`을 쓰면 NPE가 난다. `user_id` 컬럼도 schema.sql에서
NOT NULL 제약이 없으니 null 허용이 맞다.

TodoRepository에도 count()를 추가할지는 필수 2/3에서 Todo 0건을 확인할 때 필요하므로 User와 동일하게 추가:

```java
// TodoRepository.java
long count();

// JdbcTodoRepository.java
@Override
public long count() {
    return jdbcTemplate.queryForObject("select count(*) from todo", Long.class);
}
```

### 3. 조율 서비스 (필수 1)

새 패키지 `co.dingcodingco.challenge.usertodo` (또는 지호님이 선호하는 이름)에 서비스 하나:

```java
package co.dingcodingco.challenge.usertodo;

import co.dingcodingco.challenge.todo.Todo;
import co.dingcodingco.challenge.todo.TodoRepository;
import co.dingcodingco.challenge.user.User;
import co.dingcodingco.challenge.user.UserRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserTodoService {
    private final UserRepository userRepository;
    private final TodoRepository todoRepository;

    public UserTodoService(UserRepository userRepository, TodoRepository todoRepository) {
        this.userRepository = userRepository;
        this.todoRepository = todoRepository;
    }

    // 필수 2 — 트랜잭션 없음
    public Todo createUserWithTodoWithoutTransaction(String userName, String todoTitle, boolean throwAfterUserSaved) {
        User savedUser = userRepository.save(new User(userName));
        if (throwAfterUserSaved) {
            throw new TestOnlyException("intentional failure after user save");
        }
        return todoRepository.save(new Todo(savedUser.getId(), todoTitle, false));
    }

    // 필수 3 — 트랜잭션 있음
    @Transactional
    public Todo createUserWithTodo(String userName, String todoTitle, boolean throwAfterUserSaved) {
        User savedUser = userRepository.save(new User(userName));
        if (throwAfterUserSaved) {
            throw new TestOnlyException("intentional failure after user save");
        }
        return todoRepository.save(new Todo(savedUser.getId(), todoTitle, false));
    }
}
```

`throwAfterUserSaved` 같은 테스트 전용 분기를 프로덕션 서비스 메서드에 넣는 게 마음에 걸리면
대안은 있지만(예: 예외를 던지는 별도 protected 메서드를 두고 테스트에서 spy), 이번 주 범위에서는
가장 단순한 방식으로 간다 — 오버엔지니어링 피하기 (필수 3 prep-questions 판단과 동일한 원칙).

`TestOnlyException.java`:

```java
package co.dingcodingco.challenge.usertodo;

public class TestOnlyException extends RuntimeException {
    public TestOnlyException(String message) {
        super(message);
    }
}
```

### 4. 필수 2 — 트랜잭션 없는 부분 저장 테스트

**테스트 격리 원리 (필수 2, 5 공통)**

일반적인 테스트는 `@Transactional`을 붙여서 끝나면 자동 롤백시키는 방식으로 격리한다
(별도 클린징 불필요, `JdbcTodoRepositoryTest`가 쓰는 `@JdbcTest`도 이 원리). 그런데
필수 2/5는 "커밋이 실제로 됐는지/롤백됐는지" 자체가 검증 대상이라 이 방법을 못 쓴다 —
같은 트랜잭션 메커니즘을 격리 도구로도 쓰고 검증 대상으로도 쓰면 격리가 검증을 덮어버린다
(테스트용 `@Transactional`이 서비스의 커밋 여부를 가려버림). 그래서 이 두 테스트에서만
예외적으로 `@AfterEach`로 직접 delete해서 클린징한다.

**실제로 겪은 문제**: `@AfterEach`만 넣고 전체 테스트를 같이 돌리면 여전히 실패했다.
원인은 `@AfterEach`가 "이 테스트가 끝난 뒤"만 지우기 때문 — `TodoApiTest`, `TodoServiceAopTest`
같은 기존 `@SpringBootTest` 클래스들이 `todo` 테이블에 데이터를 커밋해두고 아무 정리도
안 하는데, 이 두 클래스가 `UserTodoService...Test`보다 먼저 실행되면 시작 시점부터
`count()`가 이미 오염돼 있다. 그래서 `@BeforeEach`도 추가해서 "시작 전에 남은 잔재"까지
지운다 — `@AfterEach`는 자기 자신의 뒷정리, `@BeforeEach`는 남이 남긴 것에 대한 방어.

기존 `TodoApiTest`/`TodoServiceAopTest` 자체를 고치는 게 더 근본적이지만, 그건 이번 주
범위(User/Todo 트랜잭션 미션) 밖의 기존 코드 수정이라 건드리지 않고 새 테스트 쪽에서
방어하는 쪽으로 정함.

`src/test/java/co/dingcodingco/challenge/usertodo/UserTodoServiceWithoutTransactionTest.java`
(기존 `todo` 패키지 테스트들도 main과 같은 패키지 경로에 있는 구조를 그대로 따름)

User의 id는 예외가 던져지면 서비스 메서드가 리턴을 못 하므로, `findById`가 아니라 `count()`로
검증한다 (클린징 덕에 count()가 이 테스트가 만든 것만 정확히 반영함).

```java
package co.dingcodingco.challenge.usertodo;

@SpringBootTest
class UserTodoServiceWithoutTransactionTest {

    @Autowired
    private UserTodoService userTodoService;
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private TodoRepository todoRepository;
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @BeforeEach
    void cleanUpBefore() {
        jdbcTemplate.update("delete from todo");
        jdbcTemplate.update("delete from challenge_user");
    }

    @AfterEach
    void cleanUp() {
        jdbcTemplate.update("delete from todo");
        jdbcTemplate.update("delete from challenge_user");
    }

    @Test
    void userRemainsButTodoIsNotSavedWhenExceptionThrownWithoutTransaction() {
        assertThatThrownBy(() ->
                userTodoService.createUserWithTodoWithoutTransaction("지호", "책 읽기", true)
        ).isInstanceOf(TestOnlyException.class);

        assertThat(userRepository.count()).isEqualTo(1);
        assertThat(todoRepository.count()).isEqualTo(0);
    }
}
```

`@Transactional`을 테스트 메서드에 **붙이지 않는다** — 붙이면 테스트 종료 시 전체가 롤백돼서
"User 1건 남았는지" 자체를 확인할 수 없다 (prep-questions 필수 2 판단대로). `@AfterEach`의
`delete`는 테스트가 끝난 뒤 별도로 실행되는 것이라 이 판단과 충돌하지 않는다.

### 5. 필수 3 — @Transactional 롤백 검증

`src/test/java/co/dingcodingco/challenge/usertodo/UserTodoServiceTransactionTest.java`

```java
package co.dingcodingco.challenge.usertodo;

@SpringBootTest
class UserTodoServiceTransactionTest {

    @Autowired
    private UserTodoService userTodoService;
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private TodoRepository todoRepository;
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @BeforeEach
    void cleanUpBefore() {
        jdbcTemplate.update("delete from todo");
        jdbcTemplate.update("delete from challenge_user");
    }

    @AfterEach
    void cleanUp() {
        jdbcTemplate.update("delete from todo");
        jdbcTemplate.update("delete from challenge_user");
    }

    @Test
    void userSaveIsRolledBackWhenExceptionThrownWithTransaction() {
        assertThatThrownBy(() ->
                userTodoService.createUserWithTodo("지호", "책 읽기", true)
        ).isInstanceOf(TestOnlyException.class);

        assertThat(userRepository.count()).isEqualTo(0);
        assertThat(todoRepository.count()).isEqualTo(0);
    }
}
```

`userTodoService`는 `@Autowired`로 주입받은 스프링 프록시 빈이라 `createUserWithTodo()` 호출이
프록시를 정상적으로 거친다 (self-invocation 문제 없음). count() 조회 자체도 이 테스트 메서드 안에서
새 트랜잭션으로 실행되므로(메서드 자체에 `@Transactional` 없음), 서비스 메서드의 트랜잭션이 이미
롤백된 뒤의 커밋된 상태를 보는 것이 된다 — 필수 3에서 "조회를 같은 트랜잭션 안에서 하면 안 된다"는
판단이 정확히 이 지점에 해당한다. `@AfterEach`의 cleanUp은 이 테스트가 자체적으로는 아무것도
안 남기지만(롤백됐으므로), 다른 테스트 클래스가 먼저 남긴 잔재를 없애 놓기 위한 안전장치다.

### 6. 근거형 질문 답변 반영
- 트랜잭션 경계를 서비스에 둔 이유, 체크/언체크 예외 롤백 차이, self-invocation, DB 상태 확인 이유 — prep-questions.md에 이미 판단 적어둔 내용을 실제 코드/PR 설명에 반영.

### 7. (선택) Week 2 리뷰 지적사항
- advice 호출 순서/횟수 로그 — 이번 트랜잭션 프록시 호출에도 같은 원리(self-invocation)가 적용되므로, 필수 3 테스트 작성 시 프록시를 거친 호출인지 로그로 확인하는 습관 적용할지는 지호님 판단.

## 검증 기준
1. User/Todo 저장 → 조회 → verify: 저장된 값과 조회 값 일치
2. 트랜잭션 없는 서비스, 예외 발생 → verify: User 1건, Todo 0건 (부분 저장 확인)
3. `@Transactional` 서비스, 예외 발생 → verify: User 0건, Todo 0건 (롤백 확인, 별도 조회)
4. 4번(선택 확장)은 1~3 완료 후 판단
