> `servlet-what-and-why.md`에서 "왜 Servlet이 필요한가"를 다뤘다면, 이 문서는 Servlet 인터페이스 자체를 뜯어본다.
>
> `DispatcherServlet`이 어떤 구조 위에 서 있는지, `doGet()`은 어디서 오는지가 명확해진다.

---

## Servlet 인터페이스 상속 구조

```
«interface»
Servlet
    ↑
GenericServlet    (추상 클래스, 프로토콜 독립적)
    ↑
HttpServlet       (추상 클래스, HTTP 특화)
    ↑
HttpServletBean   (Spring, 프로퍼티 바인딩)
    ↑
FrameworkServlet  (Spring, ApplicationContext 연동)
    ↑
DispatcherServlet (Spring MVC 진입점)
```

각 계층이 무엇을 추가하는지가 핵심입니다.

---

## Servlet 인터페이스 — 최상위 계약

```java
public interface Servlet {
    void init(ServletConfig config) throws ServletException;  // 초기화
    void service(ServletRequest req, ServletResponse res)     // 요청 처리
            throws ServletException, IOException;
    void destroy();                                           // 종료
    ServletConfig getServletConfig();
    String getServletInfo();
}
```

Servlet 인터페이스는 딱 세 가지만 정의합니다.

```
init()      → Servlet 인스턴스 생성 후 딱 한 번
service()   → 요청마다 호출 (스레드가 여기로 들어옴)
destroy()   → 서버 종료 시 딱 한 번
```

Tomcat은 이 인터페이스만 알면 됩니다. 구현체가 DispatcherServlet이든 다른 무엇이든 상관없이 `service()`만 호출합니다.

---

## GenericServlet — 프로토콜 독립적 기반

```java
public abstract class GenericServlet implements Servlet {

    private ServletConfig config;

    @Override
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        init();  // 하위 클래스가 오버라이드할 수 있는 편의 메서드
    }

    public void init() throws ServletException {
        // 하위 클래스가 필요하면 오버라이드
    }

    @Override
    public ServletConfig getServletConfig() {
        return config;
    }

    // service()는 여전히 추상 — 하위 클래스가 구현
    public abstract void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException;
}
```

`GenericServlet`이 추가하는 것:
- `ServletConfig` 보관
- `init(ServletConfig)` 처리 후 빈 `init()`으로 위임 — 하위 클래스가 설정 없이 초기화 로직만 작성 가능
- HTTP에 종속되지 않음 (FTP, SMTP 등 다른 프로토콜도 이론상 가능)

---

## HttpServlet — HTTP 특화

```java
public abstract class HttpServlet extends GenericServlet {

    @Override
    public void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException {

        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        service(request, response);  // HTTP 전용 service()로 위임
    }

    protected void service(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {

        // HTTP 메서드에 따라 분기
        String method = req.getMethod();
        switch (method) {
            case "GET"    -> doGet(req, res);
            case "POST"   -> doPost(req, res);
            case "PUT"    -> doPut(req, res);
            case "DELETE" -> doDelete(req, res);
            case "HEAD"   -> doHead(req, res);
            case "OPTIONS"-> doOptions(req, res);
            default       -> sendMethodNotAllowed(res);
        }
    }

    protected void doGet(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {
        // 기본 구현: 405 Method Not Allowed
        // 하위 클래스가 오버라이드해서 사용
    }

    protected void doPost(...) { ... }
    protected void doPut(...) { ... }
    protected void doDelete(...) { ... }
}
```

`HttpServlet`이 추가하는 것:
- `ServletRequest` → `HttpServletRequest` 캐스팅
- HTTP 메서드(`GET`, `POST`, `PUT`, `DELETE` 등)에 따라 `doXxx()` 분기
- 각 `doXxx()`의 기본 구현 (대부분 405 응답)

개발자는 필요한 메서드만 오버라이드하면 됩니다.

```java
// HttpServlet을 직접 쓰는 경우 (Spring 없이)
public class UserServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
            throws IOException {
        String id = req.getParameter("id");
        res.getWriter().write("{\"id\": " + id + "}");
    }
}
```

---

## Servlet 생명주기 — init → service → destroy

```
Tomcat 시작
    ↓
Servlet 인스턴스 생성 (딱 한 번)
    ↓
init(ServletConfig) 호출 (딱 한 번)
    │
    │  ← 이 시점부터 요청 처리 가능
    │
    ├─ 요청 A → service() → doGet()   (Thread-1)
    ├─ 요청 B → service() → doPost()  (Thread-2)  ← 동시 처리
    ├─ 요청 C → service() → doGet()   (Thread-3)
    │
    │  ← 서버 종료 신호
    ↓
destroy() 호출 (딱 한 번)
    ↓
인스턴스 소멸
```

**중요한 점:**
- 인스턴스는 **1개**, `service()`는 **동시에 여러 스레드**가 호출
- `init()`과 `destroy()`는 단일 스레드에서 한 번씩만 실행
- 인스턴스 변수는 모든 스레드가 공유 → 상태를 가지면 위험

---

## ServletConfig와 ServletContext

### ServletConfig — Servlet 개별 설정

```java
public interface ServletConfig {
    String getServletName();                          // Servlet 이름
    ServletContext getServletContext();               // 전체 앱 컨텍스트
    String getInitParameter(String name);            // 개별 초기화 파라미터
    Enumeration<String> getInitParameterNames();
}
```

Servlet 하나에 대한 설정을 담습니다.

```xml
<!-- web.xml에서 설정하던 방식 -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet.xml</param-value>
    </init-param>
</servlet>
```

`getInitParameter("contextConfigLocation")`으로 꺼내 씁니다.

### ServletContext — 애플리케이션 전체 공유 공간

```java
public interface ServletContext {
    String getContextPath();                          // 앱 루트 경로
    Object getAttribute(String name);                // 앱 전역 속성
    void setAttribute(String name, Object object);
    String getInitParameter(String name);            // 앱 전체 초기화 파라미터
    RequestDispatcher getRequestDispatcher(String path); // forward/include
}
```

모든 Servlet이 공유하는 공간입니다.

```
ServletContext (애플리케이션 1개에 1개)
    ├── DispatcherServlet
    ├── 다른 Servlet A
    └── 다른 Servlet B
    
→ 셋 모두 같은 ServletContext를 공유
→ setAttribute/getAttribute로 앱 전역 데이터 공유 가능
```

---

## HttpServletRequest / HttpServletResponse

Servlet이 받고 돌려주는 두 객체입니다.

### HttpServletRequest — 요청 정보 전부

```java
// 요청 라인
request.getMethod();         // "GET"
request.getRequestURI();     // "/users/123"
request.getQueryString();    // "page=1&size=20"
request.getProtocol();       // "HTTP/1.1"

// 헤더
request.getHeader("Accept");         // "application/json"
request.getHeader("Authorization");  // "Bearer token..."
request.getContentType();            // "application/json"

// 파라미터 (Query String + Form data)
request.getParameter("id");          // "123"
request.getParameterMap();           // 전체 파라미터 Map

// Body
request.getInputStream();            // Raw body 스트림
request.getReader();                 // Body를 문자열로

// 세션/쿠키
request.getSession();                // HttpSession
request.getCookies();                // Cookie[]

// 인증 (Servlet 표준 API)
request.getUserPrincipal();          // 로그인한 사용자
request.isUserInRole("ROLE_ADMIN");  // 역할 확인
```

### HttpServletResponse — 응답 작성

```java
// 상태 코드
response.setStatus(200);
response.sendError(404, "Not Found");

// 헤더
response.setContentType("application/json");
response.setHeader("Cache-Control", "no-cache");

// Body 작성
response.getWriter().write("{\"id\": 123}");   // 문자열
response.getOutputStream().write(bytes);        // 바이너리

// 리다이렉트
response.sendRedirect("/login");
```

---

## FrameworkServlet — Spring이 추가하는 것

```java
public abstract class FrameworkServlet extends HttpServletBean {

    // doGet, doPost 등을 전부 processRequest()로 위임
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        processRequest(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) {
        processRequest(request, response);  // 같은 곳으로
    }

    protected final void processRequest(HttpServletRequest request,
                                        HttpServletResponse response) {
        // 로케일, RequestAttributes 등 공통 처리
        // 최종적으로 추상 메서드 호출
        doService(request, response);
    }

    // DispatcherServlet이 구현
    protected abstract void doService(HttpServletRequest request,
                                      HttpServletResponse response);
}
```

`FrameworkServlet`이 추가하는 것:
- HTTP 메서드별 `doXxx()`를 전부 `processRequest()` 하나로 모음
- 로케일 설정, Spring 이벤트 발행 등 공통 처리
- `doService()`를 추상 메서드로 선언 → `DispatcherServlet`이 구현

그래서 `DispatcherServlet`에는 `doGet()`이 없습니다. `FrameworkServlet`에서 이미 처리하고 `doDispatch()`로 넘어옵니다.

---

## 전체 호출 체인 정리

```
Tomcat
    ↓
Servlet.service()                        ← 인터페이스
    ↓
GenericServlet.service()                 ← ServletRequest → HttpServletRequest 캐스팅
    ↓
HttpServlet.service(HttpServletRequest)  ← HTTP 메서드 분기 (GET → doGet)
    ↓
FrameworkServlet.doGet()                 ← processRequest()로 위임
    ↓
FrameworkServlet.processRequest()        ← 공통 처리 후 doService() 호출
    ↓
DispatcherServlet.doService()            ← doDispatch() 호출
    ↓
DispatcherServlet.doDispatch()           ← Spring MVC 핵심 로직
```

---

## 정리

| 클래스 | 레이어 | 핵심 역할 |
|---|---|---|
| `Servlet` | Java EE 인터페이스 | `init / service / destroy` 생명주기 계약 |
| `GenericServlet` | Java EE 추상 클래스 | `ServletConfig` 보관, 프로토콜 독립적 기반 |
| `HttpServlet` | Java EE 추상 클래스 | HTTP 메서드 분기 (`doGet`, `doPost` 등) |
| `HttpServletBean` | Spring | Spring 프로퍼티 바인딩 |
| `FrameworkServlet` | Spring | `doXxx()` → `processRequest()` 통합, ApplicationContext 연동 |
| `DispatcherServlet` | Spring MVC | `doDispatch()` — HandlerMapping, HandlerAdapter, ViewResolver 조율 |

Servlet 인터페이스를 이해하면 `DispatcherServlet.doGet()`이 어디서 오는지, `doDispatch()`가 왜 별도 메서드인지가 자연스럽게 보인다.
