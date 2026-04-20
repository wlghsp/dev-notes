> `dispatcher-servlet-flow.md`에서 전체 흐름을 파악했다면, 이 문서는 DispatcherServlet 내부 메서드를 하나씩 뜯어본다.
>
> `doDispatch()` 한 줄 한 줄이 실제로 무엇을 하는가.

---

## doDispatch() — 전체 흐름의 진입점

실제 소스코드 흐름을 단순화해서 보면 이렇습니다.

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
    
    // ① 어떤 Controller 메서드가 처리할지 찾기
    HandlerExecutionChain mappedHandler = getHandler(request);
    //  └─ HandlerExecutionChain = Controller 메서드 + 인터셉터 목록
    
    // ② 그 Controller를 어떻게 실행할지 결정
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
    
    // ③ 인터셉터 preHandle (컨트롤러 실행 전)
    // false 반환 시 컨트롤러 실행 중단 (예: 인증 실패)
    if (!mappedHandler.applyPreHandle(request, response)) {
        return;
    }
    
    // ④ 실제 Controller 메서드 실행
    ModelAndView mv = ha.handle(request, response, mappedHandler.getHandler());
    
    // ⑤ 인터셉터 postHandle (컨트롤러 실행 후)
    mappedHandler.applyPostHandle(request, response, mv);
    
    // ⑥ 응답 처리 (JSON 직렬화 or View 렌더링)
    processDispatchResult(request, response, mappedHandler, mv, ...);
}
```

`doDispatch()`는 오케스트라 지휘자 같은 역할입니다. 직접 뭔가를 하는 게 아니라 **①②③④⑤⑥ 순서대로 위임**할 뿐입니다.

---

## getHandler() — HandlerExecutionChain이 뭔가

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) {
    // 등록된 HandlerMapping들에게 순서대로 물어봄
    for (HandlerMapping mapping : this.handlerMappings) {
        HandlerExecutionChain handler = mapping.getHandler(request);
        if (handler != null) {
            return handler;  // 찾으면 바로 반환
        }
    }
    return null;
}
```

반환 타입이 `Handler`가 아니라 `HandlerExecutionChain`인 이유가 핵심입니다.

```
HandlerExecutionChain
    ├── handler          → 실제 Controller 메서드 (UserController.getUser)
    └── interceptorList  → [AuthInterceptor, LoggingInterceptor, ...]
```

**Controller만 반환하면 안 되는 이유** — 인터셉터도 함께 실행해야 하기 때문입니다. 컨트롤러와 인터셉터를 하나로 묶어서 체인 형태로 들고 다니는 객체가 `HandlerExecutionChain`입니다.

```
요청 → [AuthInterceptor.pre] → [LoggingInterceptor.pre]
     → Controller 실행
     → [LoggingInterceptor.post] → [AuthInterceptor.post]
     → 응답
```

---

## getHandlerAdapter() — 왜 어댑터가 필요한가

```java
protected HandlerAdapter getHandlerAdapter(Object handler) {
    for (HandlerAdapter adapter : this.handlerAdapters) {
        if (adapter.supports(handler)) {
            return adapter;  // 이 handler를 처리할 수 있는 어댑터 반환
        }
    }
    throw new ServletException("No adapter for handler: " + handler);
}
```

DispatcherServlet 입장에서 문제가 있습니다.

```java
// DispatcherServlet은 handler를 Object 타입으로만 알고 있음
Object handler = mappedHandler.getHandler();

// 이걸 어떻게 호출하지?
handler.invoke();  // ← 이런 메서드가 없음!!
```

Handler의 종류가 다양하기 때문입니다.

```
@RequestMapping 기반 Controller  → RequestMappingHandlerAdapter가 처리
HttpRequestHandler 구현체         → HttpRequestHandlerAdapter가 처리
Controller 인터페이스 구현체       → SimpleControllerHandlerAdapter가 처리
```

어댑터 패턴을 쓰는 이유는, **DispatcherServlet은 handler 종류를 몰라도** `adapter.handle(request, response, handler)` 한 줄로 통일해서 호출할 수 있기 때문입니다.

```java
// 어댑터가 없다면 DispatcherServlet이 직접 분기해야 함 (나쁜 코드)
if (handler instanceof HandlerMethod) {
    ((HandlerMethod) handler).invoke(...);
} else if (handler instanceof HttpRequestHandler) {
    ((HttpRequestHandler) handler).handleRequest(...);
} else if ...  // 끝도 없음

// 어댑터 패턴 덕분에 (실제 코드)
HandlerAdapter ha = getHandlerAdapter(handler);
ha.handle(request, response, handler);  // 한 줄로 끝
```

---

## processHandlerException() — 예외 처리

```java
protected ModelAndView processHandlerException(
        HttpServletRequest request, HttpServletResponse response,
        Object handler, Exception ex) throws Exception {

    // 등록된 HandlerExceptionResolver들에게 순서대로 위임
    for (HandlerExceptionResolver resolver : this.handlerExceptionResolvers) {
        ModelAndView exMv = resolver.resolveException(request, response, handler, ex);
        if (exMv != null) {
            return exMv;  // 처리됐으면 반환
        }
    }
    throw ex;  // 아무도 처리 못하면 재던짐
}
```

컨트롤러에서 예외가 발생하면 `doDispatch()`가 catch해서 이 메서드로 넘깁니다.

```
Controller에서 예외 발생
    ↓
doDispatch() catch
    ↓
processHandlerException()
    ↓
HandlerExceptionResolver 순서대로 시도
    ├── ExceptionHandlerExceptionResolver  → @ExceptionHandler 메서드 탐색
    ├── ResponseStatusExceptionResolver    → @ResponseStatus 어노테이션 처리
    └── DefaultHandlerExceptionResolver    → 스프링 표준 예외 처리 (405, 415 등)
```

**우리가 `@ExceptionHandler`를 쓰는 것도 이 흐름 위에서 동작**합니다.

```java
@ControllerAdvice  // 기본값: 전체 컨트롤러에 적용
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handle(UserNotFoundException e) {
        return ResponseEntity.status(404).body(e.getMessage());
        // ExceptionHandlerExceptionResolver가 이 메서드를 찾아 실행
    }
}

// 범위 지정도 가능
@ControllerAdvice(basePackages = "com.example.api")      // 특정 패키지만
@ControllerAdvice(assignableTypes = UserController.class) // 특정 컨트롤러만
```

---

## render() — View 렌더링

```java
protected void render(ModelAndView mv,
                      HttpServletRequest request,
                      HttpServletResponse response) throws Exception {

    // ① DispatcherServlet의 resolveViewName()으로 View 객체 요청
    //    (render() 내부에서 호출되는 별도 protected 메서드)
    View view = resolveViewName(mv.getViewName(), mv.getModel(), locale, request);

    // ② View.render() 호출 — 실제 렌더링
    view.render(mv.getModel(), request, response);
}
```

`@RestController`일 때는 `ha.handle()` 단계에서 이미 `HttpMessageConverter`가 JSON을 쓰고 응답을 완료합니다. `render()`까지 오지 않습니다.

```
@RestController (@ResponseBody 포함)
    → ha.handle() 내부에서 HttpMessageConverter가 JSON 직렬화 + 응답 완료
    → ModelAndView = null 반환
    → render() 호출 안 됨

@Controller (View 반환)
    → ha.handle()이 ModelAndView 반환 (뷰 이름 + 모델 데이터)
    → render() 호출
    → ViewResolver가 뷰 이름으로 View 객체 찾기
    → View.render()로 HTML 생성
```

**ViewResolver의 처리 방식:**

```java
// 뷰 이름 "user/profile" → 어떤 View 객체로 해석하나?
ContentNegotiatingViewResolver
    ├── Accept: text/html        → ThymeleafView ("templates/user/profile.html")
    ├── Accept: application/json → MappingJackson2JsonView
    └── Accept: text/csv         → CommaSeparatedValuesView (커스텀)
```

---

## 전체 관계 요약

```
doDispatch()                         ← 전체 흐름 조율
    │
    ├─ getHandler()                  ← "누가 처리하지?" (Controller + 인터셉터 묶음)
    │       └─ HandlerExecutionChain
    │               ├─ handler       → UserController.getUser
    │               └─ interceptors  → [Auth, Logging, ...]
    │
    ├─ getHandlerAdapter()           ← "어떻게 실행하지?" (호출 방식 추상화)
    │       └─ RequestMappingHandlerAdapter (가장 흔한 케이스)
    │
    ├─ ha.handle()                   ← 실제 Controller 실행
    │       └─ 파라미터 바인딩, 타입변환, 메서드 호출 담당
    │
    ├─ processHandlerException()     ← "예외가 났으면?" (HandlerExceptionResolver 위임)
    │       └─ @ExceptionHandler, @ResponseStatus 처리
    │
    └─ render()                      ← "응답을 어떻게 만들지?" (View 렌더링)
            └─ @RestController이면 여기까지 오지 않음 (ha.handle()에서 완료)
```
