> DispatcherServlet 흐름을 알면 자연스럽게 생기는 질문이 있다.
>
> "Controller 실행 전후에 공통 로직을 끼워 넣으려면 어디에 넣어야 하나?"
>
> 그 답이 **Filter**와 **Interceptor**다.

---

## 왜 필요한가 — 반복 코드 문제

```java
// 인증 체크를 컨트롤러마다 직접 넣는 경우
@GetMapping("/api/todos")
public List<Todo> todos(HttpServletRequest request) {
    if (request.getUserPrincipal() == null) {  // 반복
        throw new UnauthorizedException();
    }
    // ...
}

@PostMapping("/api/todos")
public void add(HttpServletRequest request, ...) {
    if (request.getUserPrincipal() == null) {  // 또 반복
        throw new UnauthorizedException();
    }
    // ...
}
```

컨트롤러가 10개면 같은 코드가 10번 들어갑니다. **Filter와 Interceptor는 이 반복을 한 곳으로 모으는 확장 포인트**입니다.

---

## 위치가 다르다 — 가장 중요한 차이

```
HTTP 요청
    ↓
[Filter]                  ← Tomcat 레벨, DispatcherServlet 바깥
    ↓
DispatcherServlet
    ↓
[Interceptor]             ← Spring MVC 레벨, DispatcherServlet 안
    ↓
Controller 실행
    ↓
[Interceptor.postHandle]
    ↓
[Filter (응답 방향)]
    ↓
HTTP 응답
```

위치가 다르면 **할 수 있는 것도 달라집니다.**

---

## Filter — Tomcat 레벨의 관문

### 기본 구조

```java
public class MyFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {

        // 요청 처리 전
        System.out.println("요청 들어옴");

        chain.doFilter(request, response);  // 다음 Filter 또는 DispatcherServlet으로

        // 요청 처리 후 (응답 방향)
        System.out.println("응답 나감");
    }
}
```

`chain.doFilter()`를 호출하면 다음 단계로 넘어가고, 호출하지 않으면 **요청을 여기서 막을 수 있습니다.**

### FilterChain — 여러 Filter가 연결된 구조

```
요청
  ↓
[EncodingFilter.doFilter()]
    chain.doFilter() 호출
        ↓
    [UserSessionFilter.doFilter()]
        chain.doFilter() 호출
            ↓
        [DispatcherServlet]  ← 최종 목적지
            ↓
        UserSessionFilter 복귀 (응답 방향)
    ↓
EncodingFilter 복귀 (응답 방향)
  ↓
HTTP 응답
```

Filter는 **양방향**입니다. `chain.doFilter()` 전은 요청 처리, 후는 응답 처리입니다.

### OncePerRequestFilter — Spring이 제공하는 편의 클래스

```java
@Component
public class UserSessionFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
                                    throws IOException, ServletException {

        var userSession = userSessionHolder.get();
        var requestWrapper = new UserSessionRequestWrapper(request, userSession);

        filterChain.doFilter(requestWrapper, response);
    }
}
```

`Filter`를 직접 구현하면 forward/include 시 필터가 여러 번 실행될 수 있습니다. `OncePerRequestFilter`는 **요청당 딱 한 번만 실행**되도록 보장합니다.

### HttpServletRequestWrapper — 요청 객체 가공

Filter에서 유용한 패턴입니다. 요청 객체를 감싸서 내용을 바꿀 수 있습니다.

```java
static class UserSessionRequestWrapper extends HttpServletRequestWrapper {

    final UserSession userSession;

    UserSessionRequestWrapper(HttpServletRequest request, UserSession userSession) {
        super(request);
        this.userSession = userSession;
    }

    @Override
    public Principal getUserPrincipal() {
        return userSession;  // 세션 정보를 표준 Servlet API에 주입
    }

    @Override
    public boolean isUserInRole(String role) {
        return userSession.getRoles().contains(role);
    }
}
```

이렇게 하면 이후 모든 계층(Interceptor, Controller)에서 `request.getUserPrincipal()`로 현재 사용자를 꺼낼 수 있습니다. **Filter가 요청에 정보를 심고, 나머지 계층이 꺼내 쓰는 패턴**입니다.

### Filter 등록

```java
// Spring Bean으로 등록하면 자동으로 Filter로 등록됨
@Component
public class UserSessionFilter extends OncePerRequestFilter { ... }

// 순서나 URL 패턴을 지정해야 할 때
@Bean
public FilterRegistrationBean<UserSessionFilter> userSessionFilter() {
    var registration = new FilterRegistrationBean<>(new UserSessionFilter());
    registration.setOrder(1);           // 실행 순서
    registration.addUrlPatterns("/*");  // 적용 URL
    return registration;
}
```

---

## Interceptor — Spring MVC 레벨의 관문

### 기본 구조

```java
public class MyInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        // Controller 실행 전
        // false 반환 시 Controller 실행 중단
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        // Controller 실행 후, View 렌더링 전
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {
        // View 렌더링까지 완료 후 (예외 발생해도 실행)
    }
}
```

### 세 메서드의 실행 시점

```
요청
  ↓
[preHandle()]          ← Controller 실행 전, false 반환 시 중단
  ↓
Controller 실행
  ↓
[postHandle()]         ← Controller 실행 후, View 렌더링 전
  ↓
View 렌더링
  ↓
[afterCompletion()]    ← 모든 처리 완료 후, 예외가 발생해도 반드시 실행
```

`afterCompletion()`은 예외 발생 시에도 실행되므로 **리소스 정리, 로깅**에 적합합니다.

### Filter와 다른 핵심 — HandlerMethod 접근

Interceptor는 DispatcherServlet 안에서 동작하므로 **어떤 Controller 메서드가 실행될지 알 수 있습니다.**

```java
public boolean preHandle(HttpServletRequest request,
                         HttpServletResponse response,
                         Object handler) throws Exception {

    if (handler instanceof HandlerMethod hm) {
        // Controller 메서드 정보 접근 가능
        String methodName = hm.getMethod().getName();
        Class<?> controllerClass = hm.getBeanType();

        // 메서드에 붙은 어노테이션 읽기 가능
        RolesAllowed rolesAllowed = hm.getMethodAnnotation(RolesAllowed.class);
    }
    return true;
}
```

Filter에서는 이게 불가능합니다. **URL은 알아도 어떤 메서드가 처리하는지는 모릅니다.**

### 인가 체크 — Interceptor가 적합한 이유

```java
public class RolesVerifyHandlerInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {

        if (handler instanceof HandlerMethod hm) {
            var rolesAllowed = hm.getMethodAnnotation(RolesAllowed.class);

            if (rolesAllowed != null) {
                // 로그인 여부 확인 (Filter에서 심어준 정보 활용)
                if (request.getUserPrincipal() == null) {
                    throw new UnauthorizedAccessException(); // 401
                }

                // 역할 확인
                var matchedRoles = Stream.of(rolesAllowed.value())
                        .filter(request::isUserInRole)
                        .collect(Collectors.toSet());

                if (matchedRoles.isEmpty()) {
                    throw new AccessDeniedException(); // 403
                }
            }
        }
        return true;
    }
}
```

컨트롤러에는 어노테이션만 붙이면 됩니다.

```java
@RolesAllowed("ROLE_USER")    // 이것만으로 인가 처리 완료
@RestController
@RequestMapping("/api/todos")
public class TodoRestController { ... }

@RolesAllowed("ROLE_ADMIN")   // 관리자만
@DeleteMapping("/api/users/{id}")
public void deleteUser(...) { ... }
```

### Interceptor 등록

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new RolesVerifyHandlerInterceptor())
                .addPathPatterns("/api/**")      // 적용 경로
                .excludePathPatterns("/api/login"); // 제외 경로
    }
}
```

---

## Filter vs Interceptor — 언제 뭘 써야 하나

| | Filter | Interceptor |
|---|---|---|
| 동작 레벨 | Tomcat (Servlet 컨테이너) | Spring MVC |
| 동작 위치 | DispatcherServlet 앞 | DispatcherServlet 안 |
| Controller 정보 접근 | 불가 | 가능 (`HandlerMethod`) |
| 어노테이션 읽기 | 불가 | 가능 |
| 요청 객체 가공 | 가능 (`RequestWrapper`) | 제한적 |
| Spring Bean 주입 | 가능 (Spring이 관리할 때) | 가능 |
| 적합한 용도 | 인코딩, 세션 초기화, 요청 가공 | 인가, 실행시간 측정, 로깅 |

**판단 기준 한 줄:**

> Controller 어노테이션을 읽어야 하면 → **Interceptor**
> 요청 객체 자체를 가공해야 하면 → **Filter**

---

## 전체 흐름에서 위치 확인

```
HTTP 요청
    ↓
[UserSessionFilter]
    └─ 세션에서 사용자 꺼내 RequestWrapper에 심기
    ↓
DispatcherServlet
    ↓
[RolesVerifyHandlerInterceptor.preHandle()]
    └─ @RolesAllowed 읽기 → 401/403 또는 통과
    └─ Filter가 심어둔 getUserPrincipal() 활용
    ↓
Controller 실행
    ↓
[RolesVerifyHandlerInterceptor.postHandle()]
    ↓
[RolesVerifyHandlerInterceptor.afterCompletion()]
    ↓
[UserSessionFilter (응답 방향)]
    ↓
HTTP 응답
```

**Filter와 Interceptor는 협력합니다.** Filter가 요청에 사용자 정보를 심고, Interceptor가 그걸 꺼내 권한을 확인하는 식으로 역할을 나눕니다.

---

## 정리

| 항목 | 설명 |
|---|---|
| **Filter** | Tomcat 레벨, DispatcherServlet 앞, 요청/응답 가공에 적합 |
| **Interceptor** | Spring MVC 레벨, Controller 정보 접근 가능, 인가/로깅에 적합 |
| **OncePerRequestFilter** | 요청당 한 번 실행 보장, Filter 구현 시 권장 |
| **HttpServletRequestWrapper** | 요청 객체를 감싸 내용 추가/변경 |
| **HandlerMethod** | Interceptor에서 Controller 메서드 정보 접근하는 객체 |
| **preHandle 반환값** | false 반환 시 Controller 실행 중단 |
| **afterCompletion** | 예외 발생 시에도 반드시 실행, 리소스 정리에 적합 |

Filter와 Interceptor를 알면 "공통 로직을 어디에 넣어야 하는가"의 답이 명확해진다.
