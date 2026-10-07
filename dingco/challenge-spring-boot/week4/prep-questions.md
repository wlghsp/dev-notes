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

네 가지 엔드포인트를 설계할 때는 UserController를 새로 만들어 역할을 분리하는 것이 훨씬 좋습니다.
POST /users와 GET /users/{userId}/todos는 UserController에 배치하고, 기존의 POST /todos와 GET /todos/{id}는 TodoController에 남겨두는 설계를 추천합니다.

- 1주차 계약이었던 `POST /todos`(user_id 없이 Todo 단독 생성)는 이번 주에 어떻게 되는가? 그대로 유지할
  것인가(레거시 엔드포인트로 남김), 아니면 없앨 것인가? 유지한다면 `user_id` 없는 Todo가 계속 생겨도
  되는가 — Week 3에서 `Todo`에 `userId` 없는 생성자를 오버로드로 남겨둔 것과 같은 결이 되는가?

> 생겨도 좋다. 

- `POST /users/{userId}/todos`는 URL의 `userId`와 Week 3에서 만든 `UserTodoService`의 흐름을 어떻게
  연결할 것인가? Week 3의 `UserTodoService`는 "User를 새로 만들고 그 id로 Todo를 만드는" 조율이었는데,
  이번 요구사항은 "이미 있는 userId로 Todo만 만드는" 것이라 다른 서비스 메서드가 필요한가?

결론부터 말씀드리면, 이미 존재하는 userId로 Todo만 생성하는 새로운 서비스 메서드가 필요합니다. Week 3에서 만드신 UserTodoService는 회원가입과 동시에 첫 Todo를 만드는 '조율(Facade)' 역할이었기 때문에, 현재 요구사항과는 목적과 흐름이 다릅니다.

## 필수 2 — 공통 오류 형식과 페이징 명세

- 현재 `ErrorResponse`는 `code`, `message` 두 필드뿐이다. `requestId`를 추가해야 하는데, 이 값은
  어디서 만들어지는가? 컨트롤러/예외 핸들러에서 매번 새로 만드는가, 아니면 필수 4번(X-Request-Id)과
  묶어서 요청 전체에 걸쳐 하나의 값을 공유하는 구조(필터, 인터셉터 등)가 필요한가? 이 둘을 따로
  구현하면 나중에 합칠 때 재작업이 생기지 않는가?

Filter 또는 Interceptor (요청 진입점) 에서 ID를 한 번만 발급하고 이를 요청 전체에서 공유해야 합니다. 

- `GET /users/{userId}/todos`의 페이징 응답(`page=0·size=20` 기본값, 최대 size, id 오름차순,
  `totalElements`·`hasNext`)은 어떤 형태의 응답 객체로 감쌀 것인가? `TodoResponse` 목록을 감싸는
  별도 `PageResponse<T>` 같은 제네릭 타입이 필요한가, 아니면 이번 엔드포인트에만 쓰는 전용 타입으로
  가는가?

GET /users/{userId}/todos 엔드포인트의 페이징 응답 객체는 프로젝트의 규모와 확장성을 고려하여 제네릭 타입인 PageResponse<T>(또는 SliceResponse<T>)로 감싸서 공통화하는 것을 추천합니다.

- "최대 size"는 몇으로 정할 것인가, 그리고 클라이언트가 그보다 큰 size를 요청하면 400인가 아니면
  최대값으로 잘라서 처리하는가? "잘못된 page·size"(예: 음수)는 검증 실패로 볼 것인가?

페이지네이션의 최대 size는 보통 100으로 정하고, 초과 요청과 음수 같은 잘못된 값은 모두 400 Bad Request로 처리하는 것이 가장 명확하고 안전합니다.

## 필수 3 — 통합 테스트로 경계값 고정

- 페이징 통합 테스트에서 "빈 목록, 여러 페이지, 안정된 정렬"을 검증하려면 각 테스트마다 얼마나 많은
  Todo를 미리 만들어둬야 하는가? Week 3에서 겪은 테스트 격리 문제(다른 테스트가 남긴 데이터와 섞임)가
  여기서는 더 크게 재발할 수 있다 — 페이징 테스트는 정확한 총 개수(`totalElements`)에 의존하기 때문이다.
  이번엔 어떤 격리 전략으로 갈 것인가? Week 3의 `@BeforeEach`/`@AfterEach` 클린징을 그대로 재사용하는가,
  아니면 이번엔 사용자별로 필터링된 목록만 보므로 다른 테스트의 잔재가 섞여도 무관한가?

테스트마다 새 User를 만들고 그 사용자의 목록만 조회한다. `totalElements`는 `where user_id = ?`로 센 사용자별 집계라서, 다른 테스트가 남긴 Todo(다른 user_id, 레거시 `POST /todos`는 null)는 섞이지 않는다. 그래서 `@BeforeEach`/`@AfterEach` 클린징은 필요 없다. 오염이 생기는 경우는 테스트들이 같은 User를 공유할 때뿐이다.

데이터 개수는 빈 목록은 0건, 여러 페이지는 5건에 size=2(3페이지, 마지막 1건, hasNext=false), 정렬은 그 5건의 오름차순 확인으로 충분하다.

> [Claude 수정 2026-10-07] 원래 답은 "사용자별 필터링만으로는 totalElements 오염을 막을 수 없으므로 기존 클린징 유지, 0/25/15개 데이터"였다. `countByUserId`가 사용자별 집계라는 점을 근거로 위처럼 바꿨다. 개수도 검증 목적에 필요한 최소(5건)로 줄였다.

- "잘못된 page·size, 검증 실패와 404"를 테스트할 때 각각 어떤 입력값을 실패 사례로 고정할 것인가?

1. 잘못된 page · size (모두 400 INVALID_REQUEST)
- page: -1, "abc" (0은 유효한 기본값이라 실패 사례가 아니다)
- size: 0, -10, "abc", 101 (최대 100의 바로 바깥 경계값)

2. 검증 실패 (400 INVALID_REQUEST)
- Todo title: "", 공백만, null, 101자 (`TodoCreateRequest`의 `@NotBlank`, `@Size(max=100)`)
- Todo completed: null
- User name: "", 101자 (`challenge_user.name`이 varchar(100)이라 검증 없이 두면 DB 오류가 난다)
- 깨진 JSON 본문

3. 404 Not Found
- `GET /todos/999999` (TODO_NOT_FOUND)
- `GET /users/999999/todos` (USER_NOT_FOUND)
- `POST /users/999999/todos` (USER_NOT_FOUND)
- id는 `Long`이라서 UUID 문자열 id는 404가 아니라 타입 불일치 400이 된다.

> [Claude 수정 2026-10-07] 원래 답은 이메일 형식, 전화번호 형식, 나이 범위, UUID형 id 등 이 API에 없는 필드였고 page=0이 실패 사례에 들어 있었다. 이 레포의 실제 필드(title, completed, name)와 엔드포인트 기준으로 다시 썼다.


## 필수 4 — X-Request-Id 처리

- "외부 X-Request-Id를 검증해 재사용하거나 새 값을 만든다"는 로직은 어디에 둘 것인가? 서블릿 필터
  (`Filter`), 스프링 `HandlerInterceptor`, 아니면 `@ControllerAdvice` 레벨 중 어느 게 맞는가?
  응답 헤더에도 같은 값을 넣어야 하니, 요청이 컨트롤러에 도달하기 전에 값을 확정해서 요청 전체
  (성공 응답, 오류 응답, 로그) 어디서나 같은 값을 꺼내 쓸 수 있는 구조가 필요해 보이는데, 이건
  스레드 로컬(MDC) 같은 걸 써야 하는가?

서블릿 필터(`Filter`)에서 한 번 정하고 MDC로 공유한다. `@ControllerAdvice`는 컨트롤러 이후에 돌고, `HandlerInterceptor`는 필터 이후, 디스패처 서블릿 안에서 돌아서 필터 단계의 로그에는 requestId를 붙일 수 없다. 응답 헤더에 넣는 것도 필터에서 먼저 해야 성공·오류 응답 모두에 같은 값이 나간다. MDC는 스레드 로컬이라 톰캣 스레드 풀에서 값이 다음 요청으로 새지 않도록 `finally`에서 `remove`해야 한다.

> [Claude 수정 2026-10-07] 이 질문의 답이 비어 있었고 바로 아래에 필수 3의 답("10~15개 데이터")이 잘못 들어가 있었다. 그 문장은 필수 3 답변으로 옮겨져 위 수정에 반영됐고(5건으로 조정), 여기에는 필터 + MDC 답을 새로 적었다.

- "검증"이라는 표현이 있다 — 외부에서 온 X-Request-Id가 어떤 조건을 만족해야 재사용하고, 어떤 경우에
  새로 만드는가? (형식이 이상하거나 비어있으면 새로 만드는 것인가?)

값이 비어있거나, 허용 문자(영숫자와 `.`, `_`, `-`)가 아닌 문자가 있거나, 길이가 1~64자를 벗어나면 재사용하지 않고 UUID를 새로 만든다. 이 조건을 만족하는 값만 재사용한다. 형식 검증이 막는 것은 개행·공백이 섞인 값이 로그에 그대로 찍히는 로그 인젝션과 아주 긴 값이다. 형식 검증은 값의 중복(서로 다른 요청이 같은 id를 보내는 경우)까지 막아주지는 못한다.

> [Claude 수정 2026-10-07] 원래 답은 "위변조 및 중복 위험이 있으면 새로 생성"이었는데, 형식 검증으로 막을 수 있는 위험이 아닌 "중복"이 근거에 들어 있었고 구체적 조건이 없었다. 허용 문자와 길이 조건을 명시하고 근거를 정정했다.

- "요청 본문·인증 정보는 로그에서 제외"해야 한다 — 지금 로깅 설정(`application.yml`)이나 앞으로 추가할
  로그 코드에서 이걸 실수로 남기기 쉬운 지점이 어디인가? (예: 예외 스택트레이스가 요청 객체를 그대로
  `toString()`해서 찍는 경우 등)

1. 예외 발생 시 객체 통째로 로깅 (가장 빈번함)
- 상황: Controller나 Service 단에서 예외가 발생했을 때, 디버깅을 편리하게 하려고 에러 로그에 요청 DTO를 그대로 넣는 경우입니다. 
- 취약점: log.error("회원가입 실패 - 요청 데이터: {}", registerDto, e); 
만약 registerDto에 별도의 마스킹 처리가 없다면, 사용자의 비밀번호, 주민등록번호, 계좌번호 등이 Plain Text로 로그에 그대로 출력됩니다.
- 이 레포 기준: Lombok을 쓰지 않고 요청 DTO(`TodoCreateRequest` 등)가 `record`다. record는 모든 필드를 `toString()`에 넣으므로, 나중에 민감 필드가 생기면 `toString()`을 직접 오버라이드해야 한다(`@ToString.Exclude`는 쓸 수 없다). 지금 DTO에는 title, completed, name뿐이라 당장의 위험은 낮다.
- 대응: 에러 로그에는 userId나 requestId 같은 비민감 식별자만 남깁니다.
- 이 레포에서 지금 실제로 위험한 지점: `HttpMessageNotReadableException`의 `getMessage()`에는 파싱 실패한 본문 조각이 들어갈 수 있다. 이를 응답이나 로그에 그대로 내보내지 않고 고정 문구를 쓴다(가이드 2번의 핸들러).
2. Spring의 HTTP 요청/응답 내장 로깅 필터 활성화 (application.yml)
- 상황: application.yml 설정만으로 HTTP 요청과 본문을 편리하게 보려고 CommonsRequestLoggingFilter 나
AbstractRequestLoggingFilter 를 Bean으로 등록하고 로그 레벨을 DEBUG 로 켜두는 경우입니다.
- 취약점: 이 필터들은 HTTP Request Body를 읽어서 로그로 출력합니다. 로그인 요청(POST /login)의 본문에 담긴 비밀번호나 결제 요청의 카드 정보가 고스란히 로그 파일에 기록됩니다.
- 대응: 운영 환경에서는 Body를 통째로 찍는 내장 필터를 절대 활성화하면 안 됩니다. 꼭 필요하다면 민감한 URL path(예: /api/v1/auth/**)를 제외하는 커스텀 필터를 구현해야 합니다.
3. 인터셉터(Interceptor) 나 AOP를 이용한 공통 로깅 (@Slf4j)
- 상황: 모든 컨트롤러의 입력 값을 모니터링하기 위해 AOP(@Before, @Around)로 JoinPoint.getArgs()를 호출해 파라미터를 일괄 로깅하는 경우입니다.
- 취약점: 로그인 메서드를 통과하는 LoginRequest 객체, 혹은 Bearer 토큰이 포함된 HttpServletRequest 객체가 그대로 문자열로 변환되어 찍힙니다. 특히 HttpServletRequest를 그대로 로그에 넣으면 헤더의 Authorization 토큰이 전부 노출될 수 있습니다.
- 대응: 로깅 AOP 적용 시, @NoLogging 같은 커스텀 어노테이션을 만들어 민감한 메서드는 로깅 대상에서 제외하거나 파라미터 타입을 체크해 필터링해야 합니다.
- 이 레포 기준: 현재 `CallCountingAspect`는 `TodoService.findById`의 호출 횟수만 세고 인자를 찍지 않아서 지금은 해당이 없다. 앞으로 로깅 AOP를 추가할 때의 위험이다.

> [Claude 수정 2026-10-07] 위 필수 4 세 번째 답변에서 오타 두 곳("경웅", "og.error")을 고치고, `@ToString` 언급이 이 레포(Lombok 미사용, record DTO)와 맞지 않아 레포 기준 설명을 추가했다. 3번 AOP 항목에는 현재 코드에서의 상태를 덧붙였다. 나머지 내용(내장 요청 로깅 필터 위험 포함)은 원문 그대로다.


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
