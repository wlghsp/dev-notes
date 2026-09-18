# Week 1 — 웹 요청에서 첫 API까지 (prep-questions)

대상 저장소: challenge-spring-boot-2026-09-wlghsp-r20
브랜치: submit/week-01__weekly-pr (홈페이지에서 생성)

미션 요구사항 원문은 missions/README.md의 "Week 1" 섹션 참고. 여기서는 실제 작업 전에 저장소 코드를 근거로 먼저 답해야 하는 질문만 정리한다. 정답을 외워 쓰지 말고, 아래 파일들을 직접 열어서 확인한 내용으로 채울 것.

확인해야 할 기존 코드:
- src/main/java/co/dingcodingco/challenge/HealthController.java
- src/main/java/co/dingcodingco/challenge/ChallengeApplication.java
- src/test/java/co/dingcodingco/challenge/ChallengeApplicationTest.java
- src/main/resources/schema.sql
- pom.xml

> DispatcherServlet을 타이핑 줄여서 DS로 표기하겠음

## 필수 1 — POST /todos: 201 / 400 계약

- 201 응답 바디에 담을 id·title·completed는 어떤 자바 타입/클래스로 표현할 것인가? (엔티티를 그대로 반환할지, 별도 응답 DTO를 둘지)

별도 응답 DTO를 사용 및 그 이유
- 엔티티 구조의 캡슐화 : 데이터베이스 테이블과 매핑되는 엔티티의 구조가 변경되더라도 API 스펙(클라이언트가 받는 데이터)은 영향을 받지 않아야 합니다.,
- 유연한 데이터 정재: 현재는 id, title, completed만 필요하지만 나중에 엔티티에 password, createdAt 같은 민감하거나 불필요한 필드가 추가되더라도 DTO를 쓰면 노출을 쉽게 막을 수 있습니다.
- 순환 참조 방지: 엔티티간의 연관관계가 복잡해질 경우, Jackson 라이브러리가 JSON으로 변환하는 과정에서 
무한 루프 (순환 참조)가 발생할 수 있습니다.
- Java 16이상을 사용한다면  record 사용

- "잘못된 입력"의 기준을 무엇으로 정할 것인가? title이 비어있거나 너무 길 때인지, completed에 이상한 값이 들어올 때인지 — 검증 규칙을 먼저 정해야 @NotBlank 같은 제약을 고를 수 있다.

  - title 비어 있거나 공백만 있으면 NotBlank 적용, 길이제한 @Size(min = 1, max = 100)
  - completed: Boolean 객체 타입 사용, @NotNull 사용으로 필수 적용

- "400 공통 오류"라고 했는데, 지금 저장소에 예외를 JSON으로 바꿔주는 공통 처리기(ControllerAdvice 등)가 이미 있는가, 없는가? HealthController와 ChallengeApplication을 직접 열어서 확인.

: 확인해봤는데 없음

- 검증 실패 시 400 응답 바디에 무엇을 담을 것인가? (필드명, 에러 메시지 등 — 최소한으로 정의)

: 필드명 에러 메시지

## 필수 2 — GET /todos/{id}: 200 / 404 계약 + 통합 테스트

- 존재하지 않는 id로 조회했을 때 404를 "공통 오류" 형식으로 준다고 했다. 이 형식은 위 400 오류와 같은 형식이어야 하는가? (하나의 공통 오류 응답 구조를 재사용하는 게 자연스러운지 판단)

: 같은 구조 사용
1. 일관성 유지: 클라이언트(프론트엔드)가 에러를 처리하는 코드를 하나로 통일할 수 있다
2. 파싱 단순화: 에러 응답에 항상 같은 필드가 오므로 에러 공통 핸들러를 쉽게 작성 가능
3. 상태코드로 구분: HTTP 상태코드가 다르므로, 클라이언트는 구조가 같아도 어떤 종류의 오류인지 구별 가능

- "세 경계"(201 성공, 400 검증 실패, 404 없음)를 하나의 테스트 클래스에서 검증할 것인가, 나눌 것인가?

: 하나의 테스트 클래스에 모아두는 것이 관리하기 좋다. 
1. 응집도 향상: 같은 API 엔드포인트나 기능 단위의 테스트가 한곳에 모여 있어서 코드를 찾기 쉽습니다.
2. 중복 제거: 공통으로 쓰는 설정이나 데이터(Setup)를 재사용하기 편합니다.
3. 빠른 파악: 클래스 하나만 열면 해당 

- ChallengeApplicationTest가 이미 @SpringBootTest + @AutoConfigureMockMvc + MockMvc를 쓰고 있다. 이번 Todo API 테스트도 같은 방식(실제 애플리케이션 컨텍스트 + MockMvc)으로 써야 하는 이유가 뭔지, "실제 애플리케이션 컨텍스트"가 왜 요구되는지 (단위 테스트로는 왜 부족한지) 스스로 판단해본다.

: 컴포넌트 간의 실제 연동과 통합 검증이 필요하기 때문입니다. 
실제 애플리케이션 컨텍스트가 필요한 이유
1. 스프링 MVC 설정 검증: 요청 매핑(@RequestMapping), JSON 직렬화/역직렬화(Jackson), 예외처리(@ExceptionHandler)등 스프링 MVC의 실제 설정이 올바르게 동작하는지 확인합니다. 
2. 보안 및 필터 연동: 시큐리티 필터 체인이나 커스텀 필터가 요청 경로에 맞게 정상적으로 작동하는지 검증합니다. 
3. 실제 빈 주입 확인: 컨트롤러가 서비스와 리포지토리 등 실제 스프링 빈들과 정상적으로 협력하고 의존성 주입에 문제가 없는지 확인합니다. 

## 필수 3 — 요청 흐름 설명 (DispatcherServlet / 검증 / HttpMessageConverter)

코드를 쓰기 전에 각 역할을 지금 아는 만큼만 짧게 적어둔다. 구현 후 본인 코드·테스트와 연결해서 evidence에 다시 쓸 것.

- DispatcherServlet이 들어온 HTTP 요청을 어떤 컨트롤러의 어떤 메서드로 연결하는지, 그 과정에서 무엇이 관여하는지

: SpringMVC에서 DispatcherServlet이 HTTP요청을 특정 컨트롤러의 메서드로 매핑하는 과정은 프레임워크의 핵심 동작 원리입니다. 
핵심적으로 관여하는 컴포넌트는 HanlderMapping, HandlerAdapter입니다. 

1. HandlerMapping을 통한 컨트롤러(메서드) 검색
요청을 받으면 DS 가 가장 먼저 HandlerMapping 인터페이스의 구현체들 순회 처리할 수 있는 핸들러 탐색 (실제 관여하는 컴포넌트 RequestMappingHandlerMapping)
- 이 컴포넌트는 애플리케이션이 구동될 때 @Controller와 @RequestMapping어노테이션이 붙은 메서드 정보를 미리 등록해 두고, 요청이 오면 매칭되는 매서드 정보(HanlderMethod)를 찾아냅니다.

2. HandlerExecutionChain 반환
HandlerMapping은 단순히 컨트롤러만 찾는 것이아니라, 해당 요청에 적용해야 할 인터셉터 목록이 포함된 HandlerExecutionChain 객체 DS에 반환

3. HandlerAdapter를 통한 실행 준비 
DispatcherServlet은 컨트롤러 메서드를 직접 호출하지 못합니다. 과거 코드 방식, 인터페이스 방식, 어노테이션 방식 등 컨트롤러의 형태가 다양할 수 있기 때문입니다. 따라서 중간에서 호출을 대행해 주는 HandlerAdapter가 관여합니다.
- 실제 관여하는 컴포넌트: RequestMappingHandlerAdapter
- 어노테이션 기반(@RequestMaping) 컨트롤러를 실행할 수 있는 어댑터가 선택됩니다.

4. 파라미터 바인딩 및 메서드 실행 (가장 중요한 변환 과정)
RequestMappingHandlerAdapter가 컨트롤러 메서드를 실행하기 직전, 메서드의 파라미터(예:@RequestParam, @RequestBody, @ModelAttribute)를 분석하고 값을 채워주기 위해 다음 두 가지가 깊게 관여합니다.
- ArgumentResolver(HandlerMethodArgumentResolver): HTTP 요청 메시지(쿼리 스트링, JSON 등)를 컨트롤러 메서드의 파라미터 타입에 맞게 변환하여 주입합니다.
- ReturnValueHandler (HandlerMethodReturnValueHandler): 컨트롤러 메서드가 반환한 결과(String, 객체 등)를 바탕으로 어떻게 응답할지 결정합니다. (@ResponseBody가 있다면 JSON으로 변환하여 응답 본문에 바로 작성)

- @RequestBody로 받은 JSON이 자바 객체로 바뀌는 지점은 어디인지, 그 변환이 실패하면 무슨 일이 일어나는지
변환되는 시점은 DS(줄여서 말하겠음) 가 컨트롤러의 메서드를 호출하기 직전
### 1.상세 동작 과정
1. DS 의 요청 수신: 클라이언트의 요청이 들어오면 DS가 이를 받습니다.
2. ArgumentResolver 작동: 컨트롤러 메서드의 파라미터들을 분석하면서 @RequestBody 를 발견하면 이를 처리할 수 있는
RequestResponseBodyMethodProcessor(HandlerMethodArgumentResolver 구현체)가 호출됩니다.
3. HttpMessageConverter 호출: 이 프로세서는 내부적으로 HTTP 메시지를 읽기 위해 HttpMessageConverter를 사용합니다.
4. Jackson 라이브러리의 역직렬화: Spring Boot 환경에서는 JSON 변환을 위해 기본적으로 MappingJackson2HttpMessageConverter 가 선택되며, 내부의 Jackson ObjectMapper가 실행되어 JSON 문자열을 자바 객체로 역직렬화합니다.

### 2. 변환이 실패하면 일어나는 일
JSON구조가 잘못되었거나 자바 객체(DTO) 매핑 조건에 맞지 않아 변환에 실패하면, Spring은 HttpMessageNotReadableException 예외를 발생시킵니다.

이 예외가 던져지면 내부적으로 다음과 같은 일이 일어납니다.

1. 클라이언트가 받는 응답
별도의 예외처리를 하지 않는다면 Spring의 DefaultHandlerExceptionResolver에 의해 기본적으로 HTTP status 400 Bad Request 상태 코드가 반환됩니다. 클라이언트는 요청이 잘못되었다는 에러 메시지를 받게 되며, 컨트롤러 메서드 내부는 실행조차 되지 않습니다.

2. 자주 발생하는 변환 실패 원인 및 세부 예외
HttpMessageNotReadableException 내부를 뜯어보면 Jackson 라이브러리가 던진 구체적인 예외 원인(nested exception)이 적혀 있습니다.

- MismatchedInputException / InvalidDefinitionException (기본 생성자 부재)
  - 원인: Jackson의 ObjectMapper가 자바 객체를 생성하려면 기본 생성자(No-Args Constructor)가 필요합니다. 기본 생성자가 없거나 파라미터가 있는 생성자만 존재할 때 에러가 발생합니다.
- UnrecognizedPropertyException (알 수 없는 필드)
  - 원인: 보내온 JSON 데이터에는 age라는 key가 존재하는데, 매핑할 자바 객체(DTO)에는 age 필드가 없을 때 발생합니다.
- JsonParseException / JsonMappingException(문법 오류 및 타입 불일치)
  - 원인: JSON 괄호가 닫히지 않았거나, 자바 DTO의 필드는 Long 타입인데 JSON으로는 "홍길동"이라는 문자열을 보냈을 때 발생합니다.

3. 실패 상황 제어하기 
실패 시 시스템이 400 Bad Request 에러 페이지를 그대로 노출하는 것을 방지하려면, @RestControllerAdvice를 활용해 예외를 깔끔한 JSON 공통 포맷으로 응답해 주는 것이 좋습니다.



- Bean Validation(@Valid 등)이 언제, 누구에 의해 실행되는지 — 컨트롤러 메서드 코드 안에서 직접 검증 로직을 호출하지 않는데도 동작하는 이유

Bean Validation은 스프링 MVC의 HandlerMethodArgumentResolver에 의해 컨트롤러 메서드가 실행되기 직전에 자동으로 동작합니다. 개발자가 직접 검증 로직을 호출하지 않아도 되는 이유는, 스프링 프레임워크가 클라이언트의 요청 데이터를 자바 갹체로 바인딩하는 과정에 검증 단계를 미리 포함시켜 두었기 때문입니다.

### 동작 원리 및 주체 

1. DispatcherServlet의 요청 수신
- 클라이언트의 요청이 들어오면 중심 컨트롤러인 DispatcherServlet이 해당 요청을 처리할 컨트롤러 메서드를 찾습니다.
2. ArgumentResolve의 개입
- 컨트롤러 메서드의 파라미터(예: @RequestBody, @ModelAttribute)를 조립하기 위해 HandlerMethodArgumentResolver가 동작합니다.
- 대표적으로 JSON 데이터를 객체로 변환할 때는 RequestResponseBodyMethodProcessor가 사용됩니다.
3. 데이터 바인딩 및 내부 검증 호출
- HTTP 요청 본문(JSON 등)을 자바 객체로 변환(바인딩)한 후, 파라미터에 @Valid 또는 @Validated 애노테이션이 붙어 있는지 확인합니다.
- 애노테이션이 존재하면, 스프링이 관리하는 DataBinder와 자바 표준 검증기인 Validator(구현체: 주로 Hibernate Validator)를 호출하여 객체 내부의 제약 조건(@NotNull, @Size 등)를 검증합니다.
4. 검증 결과 처리
- 오류 발생 시: 검증에 실패하면 MethodArgumentNotValidException (또는 BindException)이 발생하며, 컨트롤러 메서드는 아예 실행되지 않고 예외 처리기로 넘어갑니다.
- 오류가 없을 시: 검증을 통과한 유효한 객체만 컨트롤러 메서드의 파라미터로 전달되어 비즈니스 로직이 수행됩니다.


- 컨트롤러가 반환한 객체가 JSON 문자열이 되어 응답 바디에 실리는 지점 — HttpMessageConverter가 관여하는 부분

컨트롤러가 객체를 반환하면 ReturnValueHandler가 그 반환값을 어떻게 응답으로 만들지 결정합니다. @RestController가 붙어 있으면 HttpEntityMethodProcessor 같은 프로세서가 이 반환값을 뷰로 넘기지 않고 응답 본문에 직접 쓰기로 판단합니다.

### 동작 원리 및 주체
1. 컨트롤러 메서드 실행 완료
- TodoController.create() 가 TodoResponse 객체를 반환하면, RequestMappingHanlderAdapter가 이 반환값을 받습니다.
2. ReturnValueHandler의 개입
- @ResponseBody가 있는지 확인하고, 있으면 본문에 직접 쓰는 처리기를 선택합니다. 
3. HttpMessageConverter 선택
- 요청의 Accept 헤더(주로 application/json)와 반환 객체 타입을 보고, 등록된 HttpMessageConverter 중 처리 가능한 것을 찾습니다. 
- JSON이면 MappingJackson2HttpMessageConverter가 선택됩니다.
4. Jackson 직렬화
- 선택된 컨버터 내부의 Jackson ObjectMapper가 자바 객체를 JSON 문자열로 직렬화합니다.
- 이 JSON 문자열이 HTTP 응답 본문에 써지고, Content-Type: application/json이 응답 헤더에 설정됩니다.

## 다음 단계

이 파일에 답을 채운 뒤 execution-guide.md를 만들어 실제 구현 순서를 정리한다. 4번(메모리 저장소 → H2/JdbcTemplate 교체)은 선택 확장이므로 필수 1~3이 끝난 뒤에 판단한다.
