> DispatcherServlet은 Spring MVC의 핵심이지만, 많은 개발자가 그냥 "요청을 받는 것"이라고만 생각한다.
> 
> 정확히는 **"Tomcat이 받은 HTTP 요청을 Spring의 세계로 변환하고, 적절한 Controller에 전달하고, 응답을 다시 HTTP로 변환하는 중앙 관제실"**이다.

---

## DispatcherServlet이란?

DispatcherServlet은 **HttpServlet의 구현체**다. 상속 체인은 아래와 같다.

```
HttpServlet
    ↑
HttpServletBean
    ↑
FrameworkServlet
    ↑
DispatcherServlet  ← 실제 클래스
```

```java
public class DispatcherServlet extends FrameworkServlet {
    // Spring MVC의 진입점
    // Tomcat이 이 서블릿의 doGet(), doPost()를 호출
}
```

**Servlet 체인에서의 위치:**

```
HTTP 요청
    ↓
Tomcat (Servlet 컨테이너)
    ↓
DispatcherServlet (Spring MVC의 진입점, Servlet 구현체)
    ↓
Spring의 세계 (HandlerMapping, Controller, Service, etc.)
    ↓
HTTP 응답
```

---

## 요청 흐름: 상세 분석

### 1단계: HTTP 요청 도착 → DispatcherServlet 호출

```
클라이언트: GET /users/123 HTTP/1.1

Tomcat 스레드풀 → Thread-1 할당
    ↓
HttpServlet.service()             ← Tomcat이 호출
    ↓                                HTTP 메서드에 따라 분기
    ├─ GET    → doGet()
    ├─ POST   → doPost()
    ├─ PUT    → doPut()
    └─ DELETE → doDelete()
    ↓
FrameworkServlet.doGet()          ← GET 요청이므로 여기로 (다른 메서드면 각자)
    ↓
FrameworkServlet.processRequest() ← 로케일, 이벤트 등 공통 처리
    ↓
DispatcherServlet.doDispatch()    ← Spring MVC 핵심 로직 시작
```

### 2단계: HandlerMapping — 어떤 Controller인지 찾기

```java
// 개발자가 작성한 코드
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

// DispatcherServlet이 하는 일
/*
  요청: GET /users/123
  
  HandlerMapping에 물어본다:
  "이 요청 어느 Controller가 처리하지?"
  
  HandlerMapping: "UserController.getUser() 메서드가 처리합니다"
  
  (실제로는 RequestMappingHandlerMapping이 @RequestMapping 분석)
*/
```

**HandlerMapping의 종류:**

| | 설명 | 실무 사용 |
|---|---|---|
| `RequestMappingHandlerMapping` | `@GetMapping`, `@PostMapping` 등 어노테이션 기반 | ✅ 거의 모든 경우 |
| `BeanNameUrlHandlerMapping` | Bean 이름 자체가 URL (`@Component("/users")`) | ❌ Spring 초기 레거시 방식 |
| `SimpleUrlHandlerMapping` | URL 패턴을 코드로 직접 등록 | Spring 내부용 (정적 리소스 처리 등) |

실무에서는 `RequestMappingHandlerMapping` 하나만 알면 됩니다. 나머지 둘은 Spring 내부나 레거시 코드에서 간혹 보이는 수준입니다.

### 3단계: HandlerAdapter — Controller 메서드 실행

```java
/*
  DispatcherServlet: "찾은 거 맞는데, 어떻게 호출하지?"
  
  HandlerAdapter: "내가 해줄게. 파라미터 바인딩, 타입 변환까지 준비해서 넘길게"
  
  과정:
  1. @PathVariable Long id  → URL "/users/123"에서 "123" 추출, String → Long 변환
                               (URL은 텍스트 프로토콜이므로 항상 String으로 들어옴)
  2. @RequestParam int page → Query String "?page=1"에서 "1" 추출, String → int 변환
  3. @RequestBody User user → JSON body를 HttpMessageConverter가 역직렬화
                               (URL 파라미터와 달리 타입 변환이 아닌 역직렬화)
  4. 준비된 파라미터로 Controller 메서드 호출 (실제 실행은 4단계)
*/
```

**HandlerAdapter가 하는 일:**
- 요청 파라미터 추출 (`@RequestParam`, `@PathVariable`)
- 타입 변환 (String "123" → Long 123)
- Controller 메서드 호출
- 반환값 처리 — `@RestController`면 HttpMessageConverter가 JSON 직렬화 후 응답 완료 (ModelAndView = null), `@Controller`면 ModelAndView로 감싸서 DispatcherServlet에 반환

### 4단계: Controller 실행 (우리가 짠 코드)

```java
public User getUser(@PathVariable Long id) {
    // 이 코드는 Tomcat의 Thread-1에서 실행됨
    User user = userService.findById(id);  // 블로킹
    return user;
}
```

### 5단계: ViewResolver 또는 HttpMessageConverter — 응답 변환

```java
/*
  Controller에서 반환한 값을 HTTP 응답으로 변환
  
  @RestController 인 경우: (@Controller + @ResponseBody)
    → HttpMessageConverter (JSON 직렬화)
    → {"id": 123, "name": "John"}
  
  @Controller 인 경우:
    → ViewResolver (HTML 렌더링)
    → templates/user.html (Thymeleaf 등)
    // JSP는 Spring Boot에서 기본 지원하지 않음 (별도 의존성 필요)
*/
```

### 6단계: HTTP 응답 반환

```
DispatcherServlet이 응답을 가진다
    ↓
Tomcat이 HTTP 형식으로 변환
    ↓
클라이언트에게 송신
    ↓
Thread-1 반납 (ThreadPool로 돌아감)
```

---

## 전체 흐름 시각화

```
[요청 A] GET /users/123
    ↓
[Tomcat] Thread-1 할당
    ↓
[DispatcherServlet] doGet() 호출
    ↓
[HandlerMapping] /users/{id} → UserController.getUser() 매핑
    ↓
[HandlerAdapter] 파라미터 바인딩: id=123
    ↓
[UserController.getUser(123)]
    ↓
[UserService.findById(123)] — DB 조회 (블로킹)
    ↓
[User 객체 반환]
    ↓
[HttpMessageConverter] User → JSON 직렬화
    ↓
[HTTP 응답] 200 OK, {"id": 123, "name": "John"}
    ↓
[Tomcat] 클라이언트에게 송신
    ↓
[Thread-1 반납]
```

---

## DispatcherServlet의 주요 메서드

```java
public class DispatcherServlet extends FrameworkServlet {
    
    // 1. 요청 처리 진입점
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
        // 가장 중요한 메서드 — 전체 흐름이 여기서 일어남
    }
    
    // 2. Handler 찾기
    protected HandlerExecutionChain getHandler(HttpServletRequest request) {
        // HandlerMapping에 질의
        // HandlerExecutionChain = Handler + HandlerInterceptor 목록
        // 인터셉터가 함께 묶여 전달됨
    }
    
    // 3. HandlerAdapter 찾기
    protected HandlerAdapter getHandlerAdapter(Object handler) {
        // 적절한 어댑터 선택
    }
    
    // 4. 예외 처리
    protected ModelAndView processHandlerException(...) {
        // 컨트롤러에서 예외 발생 시 처리
    }
    
    // 5. 뷰 렌더링
    protected void render(ModelAndView mv, HttpServletRequest request, ...) {
        // ViewResolver로 뷰 선택, 렌더링
    }
}
```

---

## Spring Boot에서 DispatcherServlet 자동 등록

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

/*
  Spring Boot가 자동으로:
  
  1. DispatcherServlet 생성
  2. HandlerMapping 등록 (RequestMappingHandlerMapping)
  3. HandlerAdapter 등록 (RequestMappingHandlerAdapter)
  4. HttpMessageConverter 등록 (MappingJackson2HttpMessageConverter 등)
  5. ViewResolver 등록 (ContentNegotiatingViewResolver)
     → Thymeleaf/FreeMarker 의존성이 있을 때만 하위 ViewResolver 활성화
     → @RestController 위주 환경에서는 HttpMessageConverter가 핵심
  6. Tomcat에 DispatcherServlet 등록
  
  → 개발자는 @Controller만 짜면 됨
*/
```

**이전 (Spring Framework 시대):**
```xml
<!-- web.xml -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

**이제 (Spring Boot):**
```java
@SpringBootApplication  // 이 한 줄로 끝
public class Application { ... }
```

---

## 실제 예시: 요청부터 응답까지

### 시나리오
```
사용자가 브라우저에서 입력: localhost:8080/users/123
```

### 상세 흐름

```
[1] HTTP 요청 도착
GET /users/123 HTTP/1.1
Host: localhost:8080

[2] Tomcat 스레드풀에서 Thread-1 할당

[3] DispatcherServlet.doDispatch() 호출
    - request = HttpServletRequest (GET /users/123)
    - response = HttpServletResponse (아직 비어있음)

[4] HandlerMapping 실행
    RequestMappingHandlerMapping이 분석:
    - URL: /users/123
    - HTTP 메서드: GET
    - 매칭: @GetMapping("/users/{id}") 찾음
    - 결과: UserController.getUser() 메서드

[5] HandlerAdapter 실행
    RequestMappingHandlerAdapter가:
    - @PathVariable Long id → URL에서 "123"(String) 추출, Long으로 타입 변환
    - @RequestBody가 있다면 HttpMessageConverter가 JSON body 역직렬화
    - 준비된 파라미터로 Controller 메서드 호출

[6] Controller 메서드 실행
    @RestController
    public class UserController {
        @GetMapping("/users/{id}")
        public User getUser(@PathVariable Long id) {
            // id = 123L (Long 타입)
            User user = userService.findById(id);
            return user;  // User 객체 반환
        }
    }
    
    결과: User(id=123, name="John", email="john@example.com")

[7] HttpMessageConverter 실행
    Spring이 응답 객체를 JSON으로 직렬화:
    {
        "id": 123,
        "name": "John",
        "email": "john@example.com"
    }

[8] HTTP 응답 생성
    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: 54
    
    {"id":123,"name":"John","email":"john@example.com"}

[9] Tomcat이 클라이언트에게 송신

[10] Thread-1 반납 (다음 요청 대기)
```

---

## 주의사항

### DispatcherServlet은 Servlet일 뿐이다

```
Tomcat이 HTTP를 받음
    ↓
DispatcherServlet.doGet() 호출 (Servlet 인터페이스)
    ↓
그 안에서 Spring의 마법 (HandlerMapping, Controller, etc.)
```

DispatcherServlet 자체는 비즈니스 로직을 몰라요. 단지 **요청을 받아서 Spring에 넘기고, Spring의 응답을 HTTP로 변환할 뿐**입니다.

### 모든 요청이 한 번에 처리되지 않는다

```
요청 A (Thread-1)  GET /users/123     → UserController.getUser()
요청 B (Thread-2)  POST /orders       → OrderController.create()
요청 C (Thread-3)  GET /products/456  → ProductController.get()

→ 세 요청이 DispatcherServlet을 동시에 지나감 (각각 다른 스레드)
→ DispatcherServlet 인스턴스는 1개이지만 동시 처리 가능 (Servlet 설계)
```

---

## 정리

| 항목 | 설명 |
|---|---|
| **DispatcherServlet** | Spring MVC의 핵심 Servlet 구현체 |
| **역할** | Tomcat이 받은 HTTP → Spring의 HandlerMapping, Controller 호출 → HTTP 응답 |
| **HandlerMapping** | URL을 보고 어떤 Controller 메서드를 호출할지 결정 |
| **HandlerAdapter** | 파라미터 바인딩, 타입 변환, 메서드 호출을 담당 |
| **HttpMessageConverter** | 응답 객체(User, etc)를 JSON/XML로 변환 |
| **ViewResolver** | @Controller일 때 HTML 뷰를 찾아 렌더링 |
| **스레드** | 요청마다 다른 스레드에서 실행 (Tomcat ThreadPool) |

DispatcherServlet을 이해하면 "내 @Controller가 어떻게 호출되는가"가 명확해진다.
