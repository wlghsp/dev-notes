# Week 1 — 실행 가이드

prep-questions.md 답변 기준으로 실제 구현 순서를 정리한다. TDD로 진행한다 — 각 단계마다 실패하는 테스트(Red)만 미리 준비해뒀다. 구현 코드는 일부러 넣지 않았으니, 테스트를 통과시키는 최소 구현을 직접 작성해서 Green으로 만들 것. 막히면 "힌트" 부분만 참고한다.

대상 저장소: challenge-spring-boot-2026-09-wlghsp-r20
브랜치: `submit/week-01__weekly-pr` (홈페이지에서 생성 후 체크아웃)

## 전체 순서

이번 주 선택 확장(4번, 메모리 저장소 → H2·JdbcTemplate 교체)을 처음부터 반영한다. schema.sql에 이미 `todo` 테이블이 있고, 실무에서도 바로 JdbcTemplate으로 시작하는 편이 자연스럽기 때문. 필수 1~3번 요구사항(계약, 통합 테스트, 요청 흐름 설명)은 저장소 구현 방식과 무관하게 그대로 충족된다.

객체지향 관점에서 두 가지를 더 반영한다.

- **도메인에 행동을 부여한다** — `Todo`가 getter만 있는 데이터 홀더(빈약한 도메인 모델)에 머물지 않도록, 생성 규칙(제목 검증)과 상태 변경(`complete()`)을 `Todo` 자신이 책임진다.
- **저장소를 인터페이스로 둔다** — `TodoRepository`를 인터페이스로 선언하고 `JdbcTodoRepository`가 구현한다. Week 2 필수 1번("같은 인터페이스의 두 구현체 + @Qualifier/@Primary")이 이 지점을 요구하므로 미리 분리해둔다.

1. 도메인 단위 테스트 → Todo 클래스 (행동 포함)
2. 저장소 슬라이스 테스트 → 인터페이스 + JdbcTemplate 구현체
3. 요청/응답 DTO (record + Bean Validation)
4. 공통 오류 응답 클래스 + `@RestControllerAdvice`
5. 서비스 계층
6. 컨트롤러 (POST /todos, GET /todos/{id})
7. 통합 테스트 (201 / 400 / 404) — 이 테스트가 전체를 하나로 묶는다
8. 실행 검증 → evidence 기록

패키지는 기존 코드와 맞춰 `co.dingcodingco.challenge` 하위에 구성한다. 예: `co.dingcodingco.challenge.todo`.

---

## 1. 도메인 — Todo

### Red — 테스트부터 작성

`Todo` 클래스가 아직 없는 상태에서 이 테스트를 먼저 작성한다. 컴파일이 안 되는 게 정상이다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

import org.junit.jupiter.api.Test;

class TodoTest {

    @Test
    void createsWithValidTitle() {
        Todo todo = new Todo("책 읽기", false);

        assertThat(todo.getTitle()).isEqualTo("책 읽기");
        assertThat(todo.isCompleted()).isFalse();
    }

    @Test
    void rejectsBlankTitle() {
        assertThatThrownBy(() -> new Todo("   ", false))
            .isInstanceOf(IllegalArgumentException.class);
    }

    @Test
    void completeChangesStateToTrue() {
        Todo todo = new Todo("책 읽기", false);

        todo.complete();

        assertThat(todo.isCompleted()).isTrue();
    }
}
```

### Green — 직접 작성할 것

`Todo` 클래스를 만들어 위 3개 테스트를 통과시킨다. 요구사항:

- 생성자는 `title`, `completed`를 받는다. title이 null이거나 공백뿐이면 `IllegalArgumentException`을 던진다 (캡슐화 — 생성 시점부터 유효하지 않은 상태를 막는다).
- `complete()` 메서드로만 완료 상태를 바꿀 수 있다. 세터는 만들지 않는다 (SRP — 상태 변경 책임을 도메인 메서드로 응집).
- `getTitle()`, `isCompleted()`, `getId()` — 조회용 getter.
- 이후 저장소 단계에서 필요해질 것: DB에서 조회한 값(`id` 포함)으로 객체를 복원하는 경로, 그리고 저장 후 생성된 `id`를 부여하는 방법. 이 두 가지는 `TodoRepository`만 접근할 수 있어야 한다는 걸 염두에 두고 접근 제어자를 정할 것 (`public`이 아니라 패키지 프라이빗으로).

## 2. 저장소 — TodoRepository / JdbcTodoRepository

### Red — 테스트부터 작성

`@JdbcTest`는 JDBC 관련 빈만 띄우는 슬라이스 테스트라 전체 애플리케이션 컨텍스트보다 가볍다. `TodoRepository`도 `JdbcTodoRepository`도 아직 없는 상태에서 작성한다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.jdbc.core.JdbcTemplate;

@JdbcTest
@Import(JdbcTodoRepositoryTest.RepositoryConfig.class)
class JdbcTodoRepositoryTest {

    @Autowired
    private TodoRepository todoRepository;

    @Test
    void savedTodoCanBeFoundById() {
        Todo saved = todoRepository.save(new Todo("책 읽기", false));

        Todo found = todoRepository.findById(saved.getId()).orElseThrow();

        assertThat(found.getTitle()).isEqualTo("책 읽기");
    }

    @Test
    void returnsEmptyWhenNotFound() {
        assertThat(todoRepository.findById(999999L)).isEmpty();
    }

    @TestConfiguration
    static class RepositoryConfig {
        @Bean
        TodoRepository todoRepository(JdbcTemplate jdbcTemplate) {
            return new JdbcTodoRepository(jdbcTemplate);
        }
    }
}
```

### Green — 직접 작성할 것

1. `TodoRepository` 인터페이스 — `save(Todo)`, `findById(Long)` 두 메서드만 선언 (DIP + OCP — 서비스가 이 인터페이스에만 의존하게 해서, 구현체를 교체하거나 Week 2에서 두 번째 구현체를 추가해도 서비스 코드는 그대로 둔다).
2. `JdbcTodoRepository` 클래스 — `TodoRepository`를 구현하고 `JdbcTemplate`을 생성자로 주입받는다.
   - `save`: schema.sql의 `todo` 테이블(`id`, `user_id`, `title`, `completed`)에 insert한다. 이번 주는 `user_id` 없이 title·completed만 쓴다. **힌트**: auto-increment id를 즉시 받으려면 `JdbcTemplate.update(PreparedStatementCreator, KeyHolder)` 오버로드와 `GeneratedKeyHolder`를 쓴다. `Statement.RETURN_GENERATED_KEYS`를 지정해야 키가 채워진다.
   - `findById`: `jdbcTemplate.query(sql, RowMapper, id)`로 조회하고 `Optional`로 감싼다. row가 없으면 빈 `Optional`.
   - 저장 후 생성된 id를 `Todo` 객체에 채우는 방법, DB row를 `Todo`로 복원하는 방법은 1번에서 만들어둔 패키지 프라이빗 경로를 쓴다.

막히면 Spring 공식 문서에서 `JdbcTemplate` + `GeneratedKeyHolder` 예제를 검색해서 참고할 것 — 그대로 베끼지 말고 이 도메인에 맞게 옮겨 쓴다.

## 3. 요청/응답 DTO

prep-questions 답변대로 별도 DTO(record)를 쓴다.

```java
package co.dingcodingco.challenge.todo;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public record TodoCreateRequest(
    @NotBlank
    @Size(min = 1, max = 100)
    String title,

    @NotNull
    Boolean completed
) {
}
```

```java
package co.dingcodingco.challenge.todo;

public record TodoResponse(Long id, String title, boolean completed) {

    public static TodoResponse from(Todo todo) {
        return new TodoResponse(todo.getId(), todo.getTitle(), todo.isCompleted());
    }
}
```

주의: `jakarta.validation.constraints.*`를 쓴다 (`javax.*` 아님 — Spring Boot 3.x는 Jakarta EE 네임스페이스). DTO는 값 전달이 목적이라 별도 테스트 없이 바로 작성해도 된다.

## 4. 공통 오류 응답

prep-questions에서 400과 404를 같은 구조로 쓰기로 했으므로, 공통 에러 응답 클래스 하나만 만든다.

```java
package co.dingcodingco.challenge.common;

public record ErrorResponse(String code, String message) {

    public static ErrorResponse of(String code, String message) {
        return new ErrorResponse(code, message);
    }
}
```

```java
package co.dingcodingco.challenge.todo;

public class TodoNotFoundException extends RuntimeException {

    public TodoNotFoundException(Long id) {
        super("Todo not found: id=" + id);
    }
}
```

### Green — 직접 작성할 것

`GlobalExceptionHandler`를 `co.dingcodingco.challenge.common` 패키지에 `@RestControllerAdvice`로 만든다 (SRP — 예외를 HTTP 응답으로 바꾸는 책임을 컨트롤러에서 분리). 처리할 예외 세 가지:

- `MethodArgumentNotValidException` → 400. `ex.getBindingResult().getFieldErrors()`에서 첫 번째 필드 오류를 꺼내 메시지에 담는다.
- `TodoNotFoundException` → 404.
- `IllegalArgumentException` → 400. `Todo` 생성자의 검증(도메인 불변식)을 방어적으로 잡는 안전판. 정상 API 요청 경로에서는 `@Valid`가 먼저 걸러내므로 이 핸들러가 실제로 타는 일은 거의 없지만, 도메인이 직접 호출되는 다른 경로가 생기더라도 400으로 응답을 통일하기 위함이다.

세 핸들러 모두 같은 `ErrorResponse` 형태로 응답한다 (`code`, `message`).

```java
package co.dingcodingco.challenge.common;

import co.dingcodingco.challenge.todo.TodoNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

// SRP: 예외 → HTTP 응답 변환 책임을 컨트롤러에서 분리
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .findFirst()
            .map(error -> error.getField() + " " + error.getDefaultMessage())
            .orElse("invalid request");

        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("INVALID_REQUEST", message));
    }

    @ExceptionHandler(TodoNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(TodoNotFoundException ex) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("TODO_NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgument(IllegalArgumentException ex) {
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("INVALID_REQUEST", ex.getMessage()));
    }
}
```

## 5. 서비스

### Red — 테스트부터 작성

`TodoRepository`를 목(mock)으로 대체해 서비스 로직만 검증한다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

import java.util.Optional;
import org.junit.jupiter.api.Test;

class TodoServiceTest {

    private final TodoRepository todoRepository = mock(TodoRepository.class);
    private final TodoService todoService = new TodoService(todoRepository);

    @Test
    void createSavesTodoThroughRepository() {
        Todo saved = new Todo("책 읽기", false);
        when(todoRepository.save(org.mockito.ArgumentMatchers.any(Todo.class))).thenReturn(saved);

        Todo result = todoService.create("책 읽기", false);

        assertThat(result.getTitle()).isEqualTo("책 읽기");
    }

    @Test
    void findByIdThrowsWhenMissing() {
        when(todoRepository.findById(1L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> todoService.findById(1L))
            .isInstanceOf(TodoNotFoundException.class);
    }
}
```

### Green — 직접 작성할 것

`TodoService`를 `@Service`로 만들고 `TodoRepository`를 생성자로 주입받는다 (DIP — 구현체가 아니라 인터페이스를 주입받는다. Lombok 없이 생성자를 직접 쓰는 것과도 일치하고, Week 2의 생성자 주입 vs 필드 주입 비교를 준비하는 것이기도 하다).

- `create(String title, boolean completed)`: `Todo`를 만들어 저장소에 저장하고 반환.
- `findById(Long id)`: 저장소에서 조회하고, 없으면 `TodoNotFoundException`을 던진다.

## 6. 컨트롤러

DTO와 서비스가 준비됐으니 컨트롤러를 작성한다. 별도 단위 테스트 없이 바로 작성해도 된다 — 다음 단계(7번) 통합 테스트가 컨트롤러까지 한 번에 검증한다.

### Green — 직접 작성할 것

`TodoController`를 `@RestController`로 만든다 (SRP — HTTP 요청/응답 처리만 담당하고 비즈니스 로직은 서비스에 위임한다).

- `POST /todos`: `@Valid @RequestBody TodoCreateRequest`를 받아 서비스에 위임하고, `201`과 `TodoResponse`를 반환한다.
- `GET /todos/{id}`: `@PathVariable Long id`로 서비스에 위임하고, `200`과 `TodoResponse`를 반환한다.

## 7. 통합 테스트 — 세 경계를 하나로 묶는다

prep-questions 답변대로 하나의 테스트 클래스에 201/400/404 세 경계를 모은다. `ChallengeApplicationTest`와 같은 스타일(`@SpringBootTest` + `@AutoConfigureMockMvc` + `MockMvc`)을 따른다. 지금까지 만든 도메인·저장소·서비스·컨트롤러·예외 처리기가 전부 맞물려야 이 테스트가 통과한다 — TDD 사이클의 마지막 Green이자, 필수 2번이 요구하는 통합 테스트이기도 하다.

JdbcTemplate을 쓰므로 테스트가 실제 H2 DB에 값을 쓴다. 같은 테스트 클래스 안 메서드들이 컨텍스트(= 같은 인메모리 DB)를 공유해서, `createTodo_returns201WithBody`에서 저장한 row가 그대로 남는다. `findById_notFound_returns404`가 `999999L`처럼 절대 겹치지 않을 큰 id를 쓰는 이유가 여기 있다.

```java
package co.dingcodingco.challenge.todo;

import static org.hamcrest.Matchers.notNullValue;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
class TodoApiTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void createTodo_returns201WithBody() throws Exception {
        mockMvc.perform(post("/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"title": "책 읽기", "completed": false}
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(notNullValue()))
            .andExpect(jsonPath("$.title").value("책 읽기"))
            .andExpect(jsonPath("$.completed").value(false));
    }

    @Test
    void createTodo_blankTitle_returns400() throws Exception {
        mockMvc.perform(post("/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"title": "", "completed": false}
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("INVALID_REQUEST"));
    }

    @Test
    void findById_notFound_returns404() throws Exception {
        mockMvc.perform(get("/todos/{id}", 999999L))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.code").value("TODO_NOT_FOUND"));
    }

    @Test
    void findById_existing_returns200() throws Exception {
        mockMvc.perform(post("/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"title": "조회용", "completed": false}
                    """))
            .andExpect(status().isCreated());

        // 생성된 id를 응답에서 파싱해 재사용하려면 MvcResult로 받아야 함 — 직접 보완할 것
    }
}
```

`findById_existing_returns200`은 의도적으로 미완성으로 남겨뒀다. `mockMvc.perform(...).andReturn()`으로 `MvcResult`를 받아 응답 JSON에서 id를 파싱하는 방식을 직접 채워볼 것 (Jackson `ObjectMapper`로 응답 바디를 읽거나 `jsonPath`로 추출).

## 8. 실행 검증

```bash
mvn --batch-mode --no-transfer-progress test
mvn spring-boot:run
```

애플리케이션 기동 후 수동 확인 (선택):

```bash
curl -i -X POST http://localhost:8080/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "책 읽기", "completed": false}'

curl -i http://localhost:8080/todos/1
curl -i http://localhost:8080/todos/999999
```

실행 결과(터미널 출력)는 그대로 evidence-draft.md의 "검증" 섹션에 옮겨 적는다 — 수치나 성공 케이스만 고르지 말고 실제로 나온 출력을 붙여넣을 것.

## 다음 단계

1. 테스트 통과 확인 후 evidence-draft.md 작성 (변경/검증/선택 근거/근거형 질문/리뷰 반영)
2. evidence-draft를 `evidence/week-01__weekly-pr.md`로 옮김
3. PR 생성 → 자동 검사·AI 리뷰 확인 → review-notes.md 기록
