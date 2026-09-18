# Week 1 — 실행 가이드

prep-questions.md 답변 기준으로 실제 구현 순서를 정리한다. 코드는 참고용 출발점이며, 그대로 붙여넣기보다 직접 타이핑하며 각 부분이 왜 필요한지 확인할 것. 특히 필수 3번(DispatcherServlet/검증/HttpMessageConverter 최초 판단)은 prep-questions에 아직 답이 없으니, 구현하면서 채워 넣는다.

대상 저장소: challenge-spring-boot-2026-09-wlghsp-r20
브랜치: `submit/week-01__weekly-pr` (홈페이지에서 생성 후 체크아웃)

## 전체 순서

1. 도메인 클래스(엔티티) 정의
2. 저장소(Repository) — 메모리 기반으로 우선 시작
3. 요청/응답 DTO 정의 (record + Bean Validation)
4. 공통 오류 응답 클래스 + `@RestControllerAdvice`
5. 서비스 계층
6. 컨트롤러 (POST /todos, GET /todos/{id})
7. 통합 테스트 (201 / 400 / 404)
8. 실행 검증 → evidence 기록

패키지는 기존 코드와 맞춰 `co.dingcodingco.challenge` 하위에 구성한다. 예: `co.dingcodingco.challenge.todo`.

---

## 1. 도메인 — Todo.java

```java
package co.dingcodingco.challenge.todo;

public class Todo {

    private Long id;
    private final String title;
    private final boolean completed;

    public Todo(Long id, String title, boolean completed) {
        this.id = id;
        this.title = title;
        this.completed = completed;
    }

    public Long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public boolean isCompleted() {
        return completed;
    }

    void assignId(Long id) {
        this.id = id;
    }
}
```

prep-questions에서 저장소 방식(메모리 vs JdbcTemplate)을 아직 확정하지 않았다면, 이번 주 필수 범위(요청 흐름 증명)에는 메모리 저장소로 충분하다. 4번 선택 확장에서 JdbcTemplate으로 교체할 때 이 클래스가 JDBC 매핑과 충돌하지 않도록 id는 가변으로 남겨뒀다.

## 2. 저장소 — TodoRepository.java (메모리 구현)

```java
package co.dingcodingco.challenge.todo;

import java.util.Map;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;
import org.springframework.stereotype.Repository;

@Repository
public class TodoRepository {

    private final Map<Long, Todo> store = new ConcurrentHashMap<>();
    private final AtomicLong sequence = new AtomicLong(0);

    public Todo save(Todo todo) {
        long id = sequence.incrementAndGet();
        Todo saved = new Todo(id, todo.getTitle(), todo.isCompleted());
        store.put(id, saved);
        return saved;
    }

    public Optional<Todo> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }
}
```

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

주의: `jakarta.validation.constraints.*`를 쓴다 (`javax.*` 아님 — Spring Boot 3.x는 Jakarta EE 네임스페이스).

## 4. 공통 오류 응답

prep-questions에서 400과 404를 같은 구조로 쓰기로 했으므로, 공통 에러 응답 클래스 하나만 만든다. Week 4에서 `code·message·requestId`로 확장될 여지가 있지만, 이번 주는 필요한 최소 필드만 넣는다.

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

```java
package co.dingcodingco.challenge.common;

import co.dingcodingco.challenge.todo.TodoNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

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
            .body(ErrorResponse.of("INVALID_REQUEST", message));
    }

    @ExceptionHandler(TodoNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(TodoNotFoundException ex) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse.of("TODO_NOT_FOUND", ex.getMessage()));
    }
}
```

`@RestControllerAdvice`가 이번 주 필수 3번(검증이 컨트롤러 코드를 거치지 않고 동작하는 이유)의 핵심 증거다. 컨트롤러 메서드 안에 `if (title이 비었으면 400)` 같은 분기를 직접 쓰지 않아도 `@Valid` + 이 핸들러가 대신 처리한다.

## 5. 서비스

```java
package co.dingcodingco.challenge.todo;

import org.springframework.stereotype.Service;

@Service
public class TodoService {

    private final TodoRepository todoRepository;

    public TodoService(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public Todo create(String title, boolean completed) {
        Todo todo = new Todo(null, title, completed);
        return todoRepository.save(todo);
    }

    public Todo findById(Long id) {
        return todoRepository.findById(id)
            .orElseThrow(() -> new TodoNotFoundException(id));
    }
}
```

생성자 주입을 쓴다 — Lombok 없이 직접 작성하기로 한 결정과 일치하고, Week 2에서 생성자 주입 vs 필드 주입 비교를 준비하는 것이기도 하다.

## 6. 컨트롤러

```java
package co.dingcodingco.challenge.todo;

import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TodoController {

    private final TodoService todoService;

    public TodoController(TodoService todoService) {
        this.todoService = todoService;
    }

    @PostMapping("/todos")
    public ResponseEntity<TodoResponse> create(@Valid @RequestBody TodoCreateRequest request) {
        Todo todo = todoService.create(request.title(), request.completed());
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(TodoResponse.from(todo));
    }

    @GetMapping("/todos/{id}")
    public ResponseEntity<TodoResponse> findById(@PathVariable Long id) {
        Todo todo = todoService.findById(id);
        return ResponseEntity.ok(TodoResponse.from(todo));
    }
}
```

## 7. 통합 테스트

prep-questions 답변대로 하나의 테스트 클래스에 세 경계를 모은다. `ChallengeApplicationTest`와 같은 스타일(`@SpringBootTest` + `@AutoConfigureMockMvc` + `MockMvc`)을 따른다.

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
