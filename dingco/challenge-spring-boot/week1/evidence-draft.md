# Week 1 evidence 초안

evidence/week-01__weekly-pr.md로 옮길 초안. 직접 다듬어서 옮길 것.

## 변경

Todo API(POST /todos, GET /todos/{id})를 도메인-저장소-서비스-컨트롤러-예외처리 계층으로 구현했다. 저장소는 메모리가 아니라 처음부터 JdbcTemplate으로 만들어 선택 확장(4번)까지 포함시켰고, 400/404는 같은 오류 응답 구조로 통일했다.

## 검증

```
mvn --batch-mode --no-transfer-progress test
```

```
Tests run: 14, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

도메인 검증, 저장소 save/findById, 서비스 로직, API 통합 테스트(201/400/404) 전부 통과. 기존 ChallengeApplicationTest도 회귀 없음.

> 📷 **[실행 결과]** — 여기에 `mvn spring-boot:run` 후 curl로 201/400/404 확인한 결과를 캡쳐해서 붙여넣을 것. 없어도 제출 가능하지만 있으면 신뢰도가 올라간다.

## 선택 근거

- 메모리 대신 JdbcTemplate: schema.sql에 테이블이 이미 있고 실무에서도 처음부터 영속 저장소로 가는 게 자연스럽다고 판단
- 저장소를 인터페이스로 분리: Week 2가 "같은 인터페이스의 두 구현체"를 요구하므로 미리 경계를 만들어둠
- 도메인 자체 검증을 둔 이유: DTO 검증(`@NotBlank`)은 "요청이 유효한가", 도메인 검증은 "객체가 항상 유효한 상태인가"로 목적이 달라 이중으로 둠
- 아직 확인 못한 한계: 동시 요청 시 id 채번 안전성 미확인

## 근거형 질문

> 제출 시 evidence/README.md 형식(최초 판단/근거/검증 후 답변)으로 다시 풀어써야 할 수 있음. 지금은 간략 초안.

**1. HTTP 요청은 어떤 과정을 거쳐 컨트롤러 메서드의 인자가 되나요?**
DispatcherServlet → HandlerMapping → HandlerAdapter → ArgumentResolver가 파라미터를 채운다. `logging.level.org.springframework.web=DEBUG`로 확인: `RequestMappingHandlerMapping`이 `TodoController#createTodo`로 매핑하고, `RequestResponseBodyMethodProcessor`가 요청 JSON을 `TodoCreateRequest` 객체로 읽어들이는 로그를 직접 확인했다.

**2. 자바 객체는 누가 JSON 응답으로 변환하며, 그 사실을 어떻게 확인했나요?**
ReturnValueHandler → HttpMessageConverter(Jackson ObjectMapper)가 직렬화한다. 같은 DEBUG 로그에서 `HttpEntityMethodProcessor`가 `Writing [TodoResponse[...]]`를 찍는 걸 확인 — 컨트롤러가 반환한 객체가 이 지점에서 JSON으로 쓰여진다.

**3. 입력 검증 실패를 컨트롤러 코드에서 직접 분기하지 않은 이유는 무엇인가요?**
`@Valid`가 컨트롤러 진입 전에 검증을 끝낸다. 분기 코드 없이 400이 나온다는 테스트 결과가 이 판단과 일치한다.

**4. 단위 테스트가 아니라 통합 테스트로 반드시 확인해야 한 경계는 무엇이었나요?**
계층들이 실제로 맞물리는지는 단위 테스트로 못 본다. `TodoServiceTest`(mock)는 로직만 보고, `TodoApiTest`가 요청~DB 전체 경로와 `GlobalExceptionHandler`의 실제 동작을 확인했다.

**5. 현재 API 계약에서 가장 먼저 깨질 가능성이 높은 부분과 그 검증 방법은 무엇인가요?**
동시 요청 시 id 채번이 가장 먼저 깨질 것 같다. 지금까지 순차 요청만 테스트해서 확인 불가 — 다음 주차 이후 동시 저장 테스트 추가 계획.

## 리뷰 반영

자동 리뷰 수신 전
