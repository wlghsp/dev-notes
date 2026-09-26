## 변경

`challenge_user` 테이블에 대응하는 `User` 도메인과 `UserRepository`/`JdbcUserRepository`를 `Todo`와 같은 패턴(생성자 검증 + package-private id 생성자 + `assignId`, 인터페이스+`@Primary` 구현체)으로 추가했다.
`Todo`에 `userId`를 받는 생성자를 오버로드로 추가하고 `JdbcTodoRepository`의 INSERT/조회에 `user_id` 컬럼을 반영했다. 기존 2-argument 생성자와 호출부(기존 테스트 포함)는 그대로 유지했다.
User 저장과 Todo 저장을 조율하는 `UserTodoService`를 추가해, 트랜잭션이 없는 메서드(`createUserWithTodoWithoutTransaction`)와 `@Transactional`이 붙은 메서드(`createUserWithTodo`)를 같은 클래스에 나란히 두어 부분 저장 대 롤백을 대비시켰다.

## 검증

```
JAVA_HOME=<temurin-17 경로> mvn --batch-mode --no-transfer-progress test
```

Tests run: 24개 클래스 전체, Failures: 0, Errors: 0 — BUILD SUCCESS

- 필수 2 (`UserTodoServiceWithoutTransactionTest`): 트랜잭션 없는 메서드에서 User 저장 후 `TestOnlyException`을 던지게 하니 `userRepository.count()`는 1, `todoRepository.count()`는 0 — 부분 저장 확인
- 필수 3 (`UserTodoServiceTransactionTest`): `@Transactional` 메서드에서 같은 방식으로 예외를 던지니 `userRepository.count()`, `todoRepository.count()` 둘 다 0 — 롤백 확인
- Week 1~2 기존 테스트 전부 회귀 없음

참고: 처음엔 `UserTodoService...Test` 두 클래스에 `@AfterEach`로만 클린징을 넣었는데, 전체 테스트를 같이 돌리면 `count()` 검증이 실패했다. 원인은 `TodoApiTest`, `TodoServiceAopTest`처럼 `todo` 테이블에 데이터를 커밋하고 정리하지 않는 기존 테스트가 먼저 실행되면 시작 시점부터 `count()`가 오염돼 있었기 때문 — `@AfterEach`는 "이 테스트가 끝난 뒤"만 지우고 "시작 전에 남은 잔재"는 못 잡는다. `@BeforeEach`도 추가해서 해결했다.

## 선택 근거

- `UserRepository`도 `TodoRepository`와 동일하게 인터페이스+구현체 패턴 — Week 2에서 이미 다룬 패턴을 반복하는 대신, 일관성을 우선했다 (prep-questions 필수 1 판단)
- `Todo`의 기존 2-argument 생성자(`title, completed`)는 그대로 두고 `userId` 포함 생성자를 오버로드로 추가 — 기존 호출부 6곳(테스트 4개 + `TodoService.create()`)을 건드리지 않기 위함. 시그니처를 바꾸는 대안도 있었지만 범위 밖 변경이라 제외
- 트랜잭션 유/무 두 메서드를 서비스 클래스 하나에 공존 — 테스트만을 위해 클래스를 나누는 건 오버엔지니어링이라 판단 (prep-questions 필수 3 판단)
- 부분 저장/롤백 검증에 실제 `UserRepository`/`TodoRepository` 빈을 그대로 사용 — mock(`doThrow()`)을 쓰면 실제 DB 반영 여부를 볼 수 없어서 이번 검증 목적과 안 맞았다 (처음엔 mock을 고려했다가 정정함)
- 프록시 경유 호출은 `AopTestUtils` 없이 `@Autowired` 빈을 그대로 호출 — `AopTestUtils`는 프록시에서 원본을 꺼내는 용도라 오히려 반대 방향이라 제외
- 조회(`count()`)는 서비스 메서드의 트랜잭션이 끝난 뒤, 테스트 메서드 자체에는 `@Transactional`을 붙이지 않고 실행 — 같은 트랜잭션 안에서 조회하면 커밋/롤백 여부를 실제로 판단할 수 없기 때문
- 미확인 한계: `TodoApiTest`/`TodoServiceAopTest`처럼 데이터를 커밋하고 정리하지 않는 기존 테스트는 그대로 남겨뒀다 — 근본적으로는 그쪽도 클린징을 갖는 게 맞지만 이번 주 범위(User/Todo 트랜잭션 미션) 밖의 기존 코드 수정이라 손대지 않았다

## 근거형 질문

1. 트랜잭션 경계를 컨트롤러나 리포지토리가 아니라 서비스에 둔 이유는?
- 최초 판단: 하나의 비즈니스 로직이 여러 테이블 변경을 포함할 때 원자성을 보장하려면 그 흐름을 조율하는 지점, 즉 서비스에 경계를 둬야 한다고 판단했다.
- 근거: `UserTodoService.createUserWithTodo`에 `@Transactional`을 붙이고, User 저장 후 예외가 나면 Todo 저장이 도달하지 않는데도 User까지 롤백되는지를 `UserTodoServiceTransactionTest`로 확인했다.
- 검증 후 답변: 서비스 경계에 `@Transactional`을 두니 User·Todo 두 저장이 하나의 원자적 단위로 묶여 롤백됐다 — 컨트롤러(HTTP 계층)나 리포지토리(단일 테이블 단위)에 두면 이 조율이 불가능하다.

2. 체크 예외와 언체크 예외에서 기본 롤백 동작 차이는?
- 최초 판단: 스프링 `@Transactional`의 기본 정책이 언체크 예외는 롤백, 체크 예외는 커밋으로 다르게 취급할 것이라 판단했다.
- 근거: 이번 구현에서 쓴 `TestOnlyException`을 `RuntimeException`(언체크)으로 만들어 `UserTodoServiceTransactionTest`가 롤백되는 것으로 확인했다. 체크 예외 케이스는 별도로 만들어보지 않았다.
- 검증 후 답변: 언체크 예외는 기본 롤백, 체크 예외는 기본 커밋이 맞다. 체크 예외에서도 롤백하려면 `@Transactional(rollbackFor = Exception.class)`가 필요하다 — 이번 구현은 언체크 예외 케이스만 실제로 검증했다.

3. 같은 클래스 내부 호출이 트랜잭션 프록시를 우회할 수 있는 이유는?
- 최초 판단: 스프링 AOP가 프록시 방식이라, 내부 메서드 호출(`this.method()`)은 프록시를 거치지 않고 원본 객체에서 바로 실행되기 때문이라고 판단했다 — Week 2 self-invocation 답변과 같은 원리.
- 근거: 이번 구현에서는 `UserTodoService`가 자기 자신의 다른 메서드를 호출하는 구조가 아니라, 테스트가 `@Autowired` 빈을 통해 서비스 메서드를 직접 호출하는 구조라 self-invocation이 발생하지 않는다. 별도로 self-invocation을 재현하는 테스트는 만들지 않았다.
- 검증 후 답변: 원리는 Week 2와 동일하지만, 이번 구현에서는 실제로 그 우회 상황을 코드로 재현하지 않았다 — 선택 확장(4번) 대상으로 남겨둘 만한 지점.

4. 롤백 테스트가 데이터베이스 상태까지 확인해야 하는 이유는?
- 최초 판단: 예외가 던져졌다는 사실만으로는 트랜잭션이 실제로 롤백됐는지 알 수 없고, DB에 실제로 반영됐는지를 별도로 조회해야 확신할 수 있다고 판단했다.
- 근거: `UserTodoServiceTransactionTest`에서 `assertThatThrownBy`로 예외 발생만 확인하는 것에 더해, `userRepository.count()`/`todoRepository.count()`로 실제 row 개수까지 조회해서 0건임을 확인했다.
- 검증 후 답변: 예외 발생 확인만으로는 부분 커밋 여부를 알 수 없다는 게 실제로 처음 겪은 문제(격리 이슈)에서도 드러났다 — DB 상태 조회 없이는 "커밋된 다른 테스트의 잔재"와 "이번 트랜잭션이 남긴 것"을 구분할 수 없었다.

## 리뷰 반영

자동 리뷰 수신 전
