# Week 4 — 운영 가능한 API 완성 (prep-questions)

대상 저장소: challenge-spring-boot-2026-09-wlghsp-r20
브랜치: submit/week-04__weekly-pr (홈페이지에서 생성)

미션 요구사항 원문은 missions/README.md의 "Week 4" 섹션 참고. 정답을 외워 쓰지 말고, 기존 코드
(`todo/TodoController.java`, `common/GlobalExceptionHandler.java`, `common/ErrorResponse.java`,
Week 3에 만든 `UserTodoService`/`UserRepository`/`TodoRepository`)를 직접 열어서 확인한 내용으로 채울 것.

3주차 리뷰에서 "선택 확장(체크 예외, self-invocation)은 미구현"과 "기존 TodoApiTest/TodoServiceAopTest의
클린징 부재"가 다음 행동으로 지적됐다. 이번 주 범위와 무관하지만, 4주차 통합 테스트를 새로 만들 때
같은 클린징 문제가 재발하지 않게 할지 염두에 둘 것.

## 필수 1 — User–Todo 계약 완성과 1주차 계약 호환성

- 현재 `TodoController`는 `POST /todos`, `GET /todos/{id}` 두 개만 있다. 미션이 요구하는 네 엔드포인트
  (`POST /users`, `POST /users/{userId}/todos`, `GET /todos/{id}`, `GET /users/{userId}/todos`)를
  어느 컨트롤러에 놓을 것인가? `TodoController`에 다 넣을 것인가, `UserController`를 새로 만들 것인가?

>

- 1주차 계약이었던 `POST /todos`(user_id 없이 Todo 단독 생성)는 이번 주에 어떻게 되는가? 그대로 유지할
  것인가(레거시 엔드포인트로 남김), 아니면 없앨 것인가? 유지한다면 `user_id` 없는 Todo가 계속 생겨도
  되는가 — Week 3에서 `Todo`에 `userId` 없는 생성자를 오버로드로 남겨둔 것과 같은 결이 되는가?

>

- `POST /users/{userId}/todos`는 URL의 `userId`와 Week 3에서 만든 `UserTodoService`의 흐름을 어떻게
  연결할 것인가? Week 3의 `UserTodoService`는 "User를 새로 만들고 그 id로 Todo를 만드는" 조율이었는데,
  이번 요구사항은 "이미 있는 userId로 Todo만 만드는" 것이라 다른 서비스 메서드가 필요한가?

>

## 필수 2 — 공통 오류 형식과 페이징 명세

- 현재 `ErrorResponse`는 `code`, `message` 두 필드뿐이다. `requestId`를 추가해야 하는데, 이 값은
  어디서 만들어지는가? 컨트롤러/예외 핸들러에서 매번 새로 만드는가, 아니면 필수 4번(X-Request-Id)과
  묶어서 요청 전체에 걸쳐 하나의 값을 공유하는 구조(필터, 인터셉터 등)가 필요한가? 이 둘을 따로
  구현하면 나중에 합칠 때 재작업이 생기지 않는가?

>

- `GET /users/{userId}/todos`의 페이징 응답(`page=0·size=20` 기본값, 최대 size, id 오름차순,
  `totalElements`·`hasNext`)은 어떤 형태의 응답 객체로 감쌀 것인가? `TodoResponse` 목록을 감싸는
  별도 `PageResponse<T>` 같은 제네릭 타입이 필요한가, 아니면 이번 엔드포인트에만 쓰는 전용 타입으로
  가는가?

>

- "최대 size"는 몇으로 정할 것인가, 그리고 클라이언트가 그보다 큰 size를 요청하면 400인가 아니면
  최대값으로 잘라서 처리하는가? "잘못된 page·size"(예: 음수)는 검증 실패로 볼 것인가?

>

## 필수 3 — 통합 테스트로 경계값 고정

- 페이징 통합 테스트에서 "빈 목록, 여러 페이지, 안정된 정렬"을 검증하려면 각 테스트마다 얼마나 많은
  Todo를 미리 만들어둬야 하는가? Week 3에서 겪은 테스트 격리 문제(다른 테스트가 남긴 데이터와 섞임)가
  여기서는 더 크게 재발할 수 있다 — 페이징 테스트는 정확한 총 개수(`totalElements`)에 의존하기 때문이다.
  이번엔 어떤 격리 전략으로 갈 것인가? Week 3의 `@BeforeEach`/`@AfterEach` 클린징을 그대로 재사용하는가,
  아니면 이번엔 사용자별로 필터링된 목록만 보므로 다른 테스트의 잔재가 섞여도 무관한가?

>

- "잘못된 page·size, 검증 실패와 404"를 테스트할 때 각각 어떤 입력값을 실패 사례로 고정할 것인가?

>

## 필수 4 — X-Request-Id 처리

- "외부 X-Request-Id를 검증해 재사용하거나 새 값을 만든다"는 로직은 어디에 둘 것인가? 서블릿 필터
  (`Filter`), 스프링 `HandlerInterceptor`, 아니면 `@ControllerAdvice` 레벨 중 어느 게 맞는가?
  응답 헤더에도 같은 값을 넣어야 하니, 요청이 컨트롤러에 도달하기 전에 값을 확정해서 요청 전체
  (성공 응답, 오류 응답, 로그) 어디서나 같은 값을 꺼내 쓸 수 있는 구조가 필요해 보이는데, 이건
  스레드 로컬(MDC) 같은 걸 써야 하는가?

>

- "검증"이라는 표현이 있다 — 외부에서 온 X-Request-Id가 어떤 조건을 만족해야 재사용하고, 어떤 경우에
  새로 만드는가? (형식이 이상하거나 비어있으면 새로 만드는 것인가?)

>

- "요청 본문·인증 정보는 로그에서 제외"해야 한다 — 지금 로깅 설정(`application.yml`)이나 앞으로 추가할
  로그 코드에서 이걸 실수로 남기기 쉬운 지점이 어디인가? (예: 예외 스택트레이스가 요청 객체를 그대로
  `toString()`해서 찍는 경우 등)

>

## 근거형 질문 사전 판단

미션 근거형 질문 1~4에 대해, 코드를 쓰기 전 시점의 최초 판단만 짧게 적어둔다.

1. 오류 응답 형식을 공통화하면 클라이언트와 운영자에게 어떤 이점이 있나요?

>

2. 페이지 번호 기반과 커서 기반 페이징 중 현재 API에 맞는 방식을 왜 선택했나요?

>

3. 로그에 반드시 남겨야 할 정보와 남기면 안 되는 정보는 무엇인가요?

>

4. 이 API를 실제 운영하기 전에 추가할 첫 번째 관측 지표와 장애 검증은 무엇인가요?

>

## 다음 단계

이 파일에 답을 채운 뒤 execution-guide.md를 만들어 실제 구현 순서를 정리한다.
5번(선택 확장, 커서 페이징 비교 또는 관측 지표 추가)은 필수 1~4가 끝난 뒤에 판단한다.
