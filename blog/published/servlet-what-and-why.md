
> "Spring MVC는 Servlet 기반이다"는 말을 자주 듣는데, Servlet이 뭔지 모르면 Spring의 한계도 모른다.
> 
> Servlet의 핵심은 **"요청마다 프로세스를 만들지 말고, 스레드를 재사용하자"는 아이디어**다.

---

## 왜 Servlet이 필요했나?

### 1990년대: CGI의 문제점

**CGI (Common Gateway Interface)**: 웹 서버가 외부 프로그램을 호출해서 동적 콘텐츠를 만드는 방식.

```
요청 1 → 프로세스 생성 (10MB 메모리) → 처리 → 종료
요청 2 → 프로세스 생성 (10MB 메모리) → 처리 → 종료
...
요청 100 → 프로세스 생성 → 처리 → 종료

결과: 동시 접속 100명 = 프로세스 100개 = 1GB 메모리
```

**문제**: 요청마다 프로세스를 만드는 건 너무 비싸다.

### 1997년: Servlet의 등장

```
아이디어: 프로세스 대신 스레드를 재사용하자

서버 시작
    ↓
JVM 프로세스 1개 실행
    ↓
스레드 200개 준비해두기 (스레드풀)
    ↓
요청 1 → 스레드 1 할당 → 처리 → 반납
요청 2 → 스레드 2 할당 → 처리 → 반납
...
요청 100 → 스레드 100 할당 → 처리 → 반납

결과: 동시 접속 100명 = 스레드 100개 = 100MB 메모리
```

**장점**: 프로세스(1개 = 10MB)가 아니라 스레드(1개 = 1MB) 사용 → 10배 효율

---

## 그럼 Servlet이 뭐지?

Servlet은 Java 표준 인터페이스다. 즉, **계약**이다.

"GET 요청이 들어오면 `doGet()` 호출할게. POST면 `doPost()` 호출하고"

개발자는 이 약속을 지키고 구현하면 된다:

```java
public class UserServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        String id = req.getParameter("id");
        User user = userRepository.findById(id);
        res.getWriter().write(convertToJson(user));
    }
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        // POST 처리
    }
}
```

HTTP 메서드별로 `doGet()`, `doPost()`, `doPut()`, `doDelete()` 등이 있다. 서블릿 컨테이너가 요청의 HTTP 메서드를 보고 자동으로 맞는 메서드를 호출한다.

---

## Servlet의 핵심 구조

### 1. Servlet 인스턴스는 한 번만 만들어진다

서버 시작 → Servlet 인스턴스 생성 (init) → 요청 처리 (service) → 요청 처리 → ... → 종료 (destroy)

즉, 여러 요청이 **같은 Servlet 인스턴스**를 공유한다.

### 2. 각 요청마다 다른 스레드에서 처리된다

요청 1 → Thread-1에서 doGet() 실행  
요청 2 → Thread-2에서 doGet() 실행 (동시)

**주의**: 같은 인스턴스이므로 인스턴스 변수는 위험하다.

```java
// ❌ 위험
public class Bad extends HttpServlet {
    private String userId;  // 모든 요청이 공유
    
    protected void doGet(...) {
        this.userId = req.getParameter("id");
        // Thread-2가 다른 값으로 덮어쓸 수 있음!
    }
}

// ✅ 안전
public class Good extends HttpServlet {
    protected void doGet(...) {
        String userId = req.getParameter("id");  // 로컬 변수
    }
}
```

---

## Servlet 컨테이너는 Tomcat 안에 있다

### 구조

```
Tomcat (WAS, 웹 서버)
└── Servlet 컨테이너 (HTTP 요청 받아서 Servlet 호출)
    ├── DispatcherServlet (Spring MVC)
    ├── 다른 Servlet들
    └── ...
```

**정확히:**
- **Servlet 컨테이너 = Tomcat의 일부**
- **Spring = Tomcat 위에 올라온 프레임워크**
- Spring은 Servlet 컨테이너에 DispatcherServlet을 등록하고, 모든 요청이 여기를 거쳐 @Controller로 라우팅된다

### Servlet 컨테이너가 하는 일

HTTP 요청이 들어오면:

```
1. HTTP 파싱 및 URL 매핑
2. 스레드풀에서 스레드 할당
3. Servlet.doGet() 또는 doPost() 호출
4. 응답 반환
5. 스레드 반납
```

개발자는 3번만 구현한다. 나머지는 Tomcat이 한다.

### Spring Boot에서의 자동 설정

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

이 한 줄로:
1. Tomcat 시작
2. DispatcherServlet 자동 생성 및 등록
3. localhost:8080 대기

**이전**엔 개발자가 web.xml이나 Java Config로 수동 등록했다. **Spring Boot는 자동화했다.**

---

## 실제 흐름: Tomcat → Spring → Tomcat

```
클라이언트가 GET /users/123 요청
    ↓
Tomcat (Servlet 컨테이너)
    1. HTTP 요청 파싱
    2. 스레드풀에서 스레드 할당 (예: Thread-1)
    3. DispatcherServlet 호출
    ↓
DispatcherServlet (Spring MVC)
    1. URL 분석 (/users/123)
    2. @Controller 찾기 (UserController)
    3. @GetMapping 찾기 (getUser 메서드)
    4. 메서드 실행
    ↓
UserController.getUser()
    1. DB 조회 (Thread-1이 I/O 기다림)
    2. 데이터 처리
    3. JSON 응답 객체 생성
    4. 반환
    ↓
DispatcherServlet이 응답 객체를 받아서
    1. JSON으로 변환
    2. HTTP 응답으로 변환
    ↓
Tomcat이 응답을 클라이언트에게 송신
    ↓
Thread-1 반납 (스레드풀로 돌아감)
```

**핵심**: 
- Tomcat이 시작 (HTTP 받음)
- Spring은 Tomcat이 호출한 DispatcherServlet 위에서 일함
- 응답도 Tomcat이 최종 송신

즉, **Tomcat이 Spring을 감싸고 있다**.

개발자는 `@RestController`만 짜면 된다:

```java
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // 이 코드는 Tomcat의 Thread-1 안에서 실행된다
        return userRepository.findById(id);  // 블로킹
    }
}
```

내부적으로는 여전히:
- Tomcat이 스레드 할당
- 스레드가 I/O 기다리는 동안 묶여 있음 (블로킹)
- 동시 접속 수가 많으면 메모리 문제 발생

---

## 정리

**Servlet이란?**
- Java 웹의 표준 인터페이스
- "요청 = 스레드 1개" 모델
- CGI(프로세스 방식)의 대안으로 1997년 등장

**Servlet 컨테이너 = WAS (Tomcat, Jetty, Undertow)**
- HTTP 요청을 받아서 Servlet을 호출하는 프로그램
- 스레드풀 관리
- 개발자는 비즈니스 로직만 작성

**Spring MVC**
- Servlet 위에 얹힌 프레임워크
- DispatcherServlet이 모든 요청을 받음
- @Controller, @RestController로 간편하게 작성 가능

Servlet을 이해하면 Spring과 Java 웹 개발의 기초가 보인다.
