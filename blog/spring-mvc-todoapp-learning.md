# 스프링을 쓰고 있지만 스프링을 모른다면 — Spring MVC 교육 정리

> 스프링으로 개발은 하고 있지만, 스프링이 뒤에서 무엇을 해주는지 모른 채 쓰는 경우가 많습니다.  
> 이 문서는 `wlghsp/todoapp` 실습을 통해 "스프링에는 이런 기능이 있고, 이렇게 간단하게 쓸 수 있다"는 것을 보여주기 위해 진행한 교육 내용을 정리한 것입니다.

---

## 들어가며 — 왜 이걸 배워야 할까

스프링을 처음 배우면 대부분 이렇게 시작합니다.

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser(@RequestParam String id) { ... }
}
```

`@RequestParam`이 왜 동작하는지, `@RestController`가 어떻게 JSON을 돌려주는지 모른 채로 "그냥 되니까" 씁니다.

이게 문제가 되는 순간은 **요구사항이 조금만 달라졌을 때**입니다.

- "모든 컨트롤러에 로그인 사용자 정보를 넘겨야 해"
- "특정 API는 관리자만 접근 가능하게 해야 해"
- "CSV 다운로드도 지원해야 해"

이런 요구사항이 생기면, 스프링이 이미 만들어둔 확장 포인트를 모르는 사람은 **컨트롤러마다 반복 코드를 넣거나**, 억지로 복잡한 방법을 찾습니다.

스프링은 이미 다 준비해뒀습니다. 우리가 그걸 몰랐을 뿐입니다.

---

## 실습 프로젝트 구조

교육에 사용한 TodoApp은 간단한 할 일 관리 앱이지만, Spring MVC의 핵심 기능을 모두 담고 있습니다.

```
todoapp/
├── core/        ← 순수 비즈니스 로직 (Spring 의존 없음)
├── data/        ← DB/저장소 구현체
├── security/    ← 인증·인가 도메인
└── web/         ← Spring MVC 레이어
    ├── config/  ← MVC 설정
    └── support/ ← 커스텀 확장 컴포넌트
```

---

## 1. 설정값 관리 — 흩어진 `@Value`를 하나로 모으기

### 대부분 이렇게 씁니다

```java
@Service
public class SomeService {
    @Value("${app.author}") private String author;
    @Value("${app.description}") private String description;
    @Value("${app.max-size}") private int maxSize;
}
```

설정이 늘어날수록 `@Value`가 여기저기 흩어지고, 오타가 나도 런타임에서야 알게 됩니다.

### 스프링이 준비한 방법

```yaml
# application.yml
todoapp:
  site:
    author: SpringRunner
    description: What are your plans today?
```

```java
@ConfigurationProperties("todoapp.site")
public class SiteProperties {
    private String author = "unknown";
    private String description = "TodoApp templates for Server-side";
    // getter/setter
}
```

설정값을 하나의 객체로 묶어 관리합니다. 이 객체를 Bean으로 등록하면 어디서든 주입받아 씁니다.

**달라지는 것:**
- IDE 자동완성으로 오타 방지
- 관련 설정이 한 곳에 모임
- 기본값을 코드로 명시할 수 있음

---

## 2. 공통 데이터 처리 — 모든 뷰에 반복해서 넣지 않기

### 대부분 이렇게 씁니다

```java
@GetMapping("/todos")
public String todos(Model model) {
    model.addAttribute("site", siteProperties);  // 매번 추가
    model.addAttribute("todos", todoService.findAll());
    return "todos";
}

@GetMapping("/profile")
public String profile(Model model) {
    model.addAttribute("site", siteProperties);  // 또 추가
    return "profile";
}
```

헤더나 푸터에 사이트 정보가 필요하다면, 모든 컨트롤러에 같은 코드가 반복됩니다.

### 스프링이 준비한 방법

```java
@ControllerAdvice
public class GlobalControllerAdvice {

    @ModelAttribute("site")
    public SiteProperties siteProperties() {
        return siteProperties;  // 모든 요청에 자동으로 "site" 모델 추가
    }
}
```

`@ControllerAdvice` 안의 `@ModelAttribute` 메서드는 **모든 컨트롤러 요청 전에 자동 실행**됩니다. 이제 개별 컨트롤러에서 `site`를 넣을 필요가 없습니다.

```html
<!-- 모든 Thymeleaf 템플릿에서 그냥 쓸 수 있음 -->
<footer th:text="${site.author}"></footer>
```

---

## 3. 인증 — Spring Security 없이 직접 만들어보기

Spring Security를 먼저 배우면 내부 동작을 이해하기 어렵습니다. 직접 만들어보면 "Spring Security가 이걸 대신 해주는구나"가 보입니다.

### 로그인 처리

```java
@PostMapping("/login")
public String loginProcess(@Valid LoginCommand command, BindingResult bindingResult, Model model) {

    // 입력값 검증 실패
    if (bindingResult.hasErrors()) {
        model.addAttribute("message", "너 실수했음!!!");
        return "login";
    }

    User user;
    try {
        user = verifyUserPassword.verify(command.username(), command.password());
    } catch (UserNotFoundException e) {
        user = registerUser.register(command.username(), command.password());
    } catch (UserPasswordNotMatchedException e) {
        model.addAttribute("message", "비밀번호가 달라요");
        return "login";
    }

    userSessionHolder.set(new UserSession(user));
    return "redirect:/todos";  // PRG 패턴
}

record LoginCommand(
    @Size(min = 4, max = 20) String username,
    String password
) {}
```

**포인트:**
- `@Valid`와 `BindingResult`로 입력값 검증을 선언적으로 처리
- `redirect:` 접두어 → POST 후 GET 리다이렉트 (새로고침해도 중복 제출 안 됨)

### 세션을 요청 객체에 담기 — Filter

로그인 후 매 요청마다 "지금 누가 요청하는가"를 알아야 합니다. 이걸 Filter로 처리합니다.

```java
@Component
public class UserSessionFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws IOException, ServletException {
        var userSession = userSessionHolder.get();

        // 표준 서블릿 API(getUserPrincipal)에 맞게 래핑
        var requestWrapper = new UserSessionRequestWrapper(request, userSession);

        filterChain.doFilter(requestWrapper, response);
    }

    static class UserSessionRequestWrapper extends HttpServletRequestWrapper {
        final UserSession userSession;

        @Override
        public Principal getUserPrincipal() {
            return userSession;
        }

        @Override
        public boolean isUserInRole(String role) {
            return userSession.getRoles().contains(role);
        }
    }
}
```

`HttpServletRequestWrapper`로 요청 객체를 감싸서 `getUserPrincipal()`을 오버라이드합니다. 이후 모든 계층에서 표준 서블릿 API로 현재 사용자를 꺼낼 수 있습니다.

> `OncePerRequestFilter`를 쓰는 이유: forward/include가 발생해도 필터가 딱 한 번만 실행되도록 보장합니다.

---

## 4. 인가 — 반복 코드 없이 접근 제어하기

### 대부분 이렇게 씁니다

```java
@GetMapping("/api/todos")
public List<Todo> todos(HttpServletRequest request) {
    if (request.getUserPrincipal() == null) {
        throw new UnauthorizedException();
    }
    // ...
}

@PostMapping("/api/todos")
public void add(...) {
    if (request.getUserPrincipal() == null) {  // 또 같은 코드
        throw new UnauthorizedException();
    }
    // ...
}
```

### 스프링이 준비한 방법 — Interceptor

```java
public class RolesVerifyHandlerInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        if (handler instanceof HandlerMethod hm) {
            var rolesAllowed = hm.getMethodAnnotation(RolesAllowed.class);

            if (Objects.nonNull(rolesAllowed)) {
                // 로그인 여부 확인
                if (Objects.isNull(request.getUserPrincipal())) {
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

이 인터셉터를 등록하면 컨트롤러에서는 어노테이션만 붙이면 됩니다.

```java
@RolesAllowed("ROLE_USER")   // 이것만으로 인가 처리 완료
@RestController
@RequestMapping("/api/todos")
public class TodoRestController { ... }
```

**Filter와 Interceptor의 차이:**

| | Filter | Interceptor |
|---|---|---|
| 동작 위치 | DispatcherServlet 앞 | DispatcherServlet 안 |
| 컨트롤러 정보 접근 | 불가 | 가능 (`HandlerMethod`) |
| 어노테이션 읽기 | 불가 | 가능 |
| 적합한 용도 | 인코딩, 세션 초기화 | 인가, 실행시간 측정 |

인가 체크에 인터셉터를 쓰는 이유가 여기 있습니다. **컨트롤러 메서드의 어노테이션을 읽을 수 있기 때문**입니다.

---

## 5. 커스텀 파라미터 주입 — `@RequestParam` 너머의 세계

### 이런 불편함에서 시작합니다

```java
@GetMapping("/profile")
public String profile(HttpServletRequest request, Model model) {
    // 컨트롤러마다 세션에서 직접 꺼내야 함
    var userSession = (UserSession) request.getSession().getAttribute("userSession");
    model.addAttribute("user", userSession.getUser());
    return "profile";
}
```

### 스프링이 준비한 방법 — ArgumentResolver

`@RequestParam`, `@PathVariable`, `@AuthenticationPrincipal` 같은 어노테이션들이 동작하는 원리가 바로 이 `HandlerMethodArgumentResolver`입니다.

```java
public class UserSessionHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        // "이 파라미터 타입을 내가 처리할 수 있나요?"
        return UserSession.class.isAssignableFrom(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ...) {
        // "그럼 이 값을 주입하세요"
        return userSessionHolder.get();
    }
}
```

등록은 `WebMvcConfigurer`에서:

```java
@Override
public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    resolvers.add(new UserSessionHandlerMethodArgumentResolver(userSessionHolder));
}
```

이제 컨트롤러에서는 파라미터 타입만 선언하면 자동으로 주입됩니다:

```java
@GetMapping("/profile")
public String profile(UserSession userSession, Model model) {
    // 세션을 꺼내는 코드가 사라짐
    model.addAttribute("user", userSession.getUser());
    return "profile";
}
```

---

## 6. 커스텀 응답 처리 — 리턴 타입으로 응답 방식 결정하기

### 이런 요구사항이 생기면

"프로필 이미지 API를 만들어야 하는데, 이미지 파일 자체를 응답으로 보내야 해."

```java
// 이렇게 하면 되긴 하는데...
@GetMapping("/profile-picture")
public void getProfilePicture(HttpServletResponse response) throws IOException {
    var picture = profilePictureStorage.load(...);
    picture.getInputStream().transferTo(response.getOutputStream());
}
```

응답 처리 로직이 컨트롤러 안에 섞입니다.

### 스프링이 준비한 방법 — ReturnValueHandler

```java
public class ProfilePictureReturnValueHandler implements HandlerMethodReturnValueHandler {

    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        // "이 리턴 타입을 내가 처리할 수 있나요?"
        return ProfilePicture.class.isAssignableFrom(returnType.getParameterType());
    }

    @Override
    public void handleReturnValue(Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest) throws Exception {
        var response = webRequest.getNativeResponse(HttpServletResponse.class);
        var profilePicture = profilePictureStorage.load(((ProfilePicture) returnValue).getUri());
        profilePicture.getInputStream().transferTo(response.getOutputStream());

        mavContainer.setRequestHandled(true);  // View 렌더링 건너뜀
    }
}
```

이제 컨트롤러는 `ProfilePicture` 객체만 반환하면 됩니다:

```java
@GetMapping("/profile-picture")
public ProfilePicture getProfilePicture(UserSession userSession) {
    return userSession.getUser().getProfilePicture();  // 반환만 하면 끝
}
```

**ArgumentResolver와의 관계:**

```
요청 들어옴 → [ArgumentResolver] 파라미터 채워줌 → 컨트롤러 실행 → [ReturnValueHandler] 응답 처리
```

입력(파라미터)과 출력(응답)을 각각 커스터마이징하는 두 확장 포인트입니다.

---

## 7. 콘텐츠 협상 — 같은 URL, 다른 포맷

### 이런 요구사항이 생기면

"Todo 목록을 웹에서는 HTML로, API 클라이언트는 JSON으로, 엑셀 담당자는 CSV로 받고 싶다."

URL은 `/todos`로 같은데 응답 형식만 다르게 하고 싶은 상황입니다.

### 스프링이 준비한 방법 — ContentNegotiatingViewResolver

클라이언트가 요청할 때 `Accept` 헤더로 원하는 포맷을 알려줍니다.

```
GET /todos  Accept: text/html    → HTML 응답
GET /todos  Accept: application/json → JSON 응답
GET /todos  Accept: text/csv    → CSV 파일 다운로드
```

CSV는 스프링 기본 지원이 아니므로 커스텀 View를 만들어 등록합니다:

```java
public class CommaSeparatedValuesView extends AbstractView {

    public CommaSeparatedValuesView() {
        setContentType("text/csv");
    }

    @Override
    protected void renderMergedOutputModel(Map<String, Object> model,
                                           HttpServletRequest request,
                                           HttpServletResponse response) throws Exception {
        response.setHeader(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"todos.csv\"");

        var spreadsheet = Spreadsheet.obtainSpreadsheet(model);
        for (var row : spreadsheet.getRows()) {
            response.getWriter().println(row.joining(","));
        }
    }
}
```

```java
// ContentNegotiatingViewResolver에 커스텀 View 추가
@Autowired
public void configurer(ContentNegotiatingViewResolver viewResolver) {
    var defaultViews = new ArrayList<>(viewResolver.getDefaultViews());
    defaultViews.add(new CommaSeparatedValuesView());
    viewResolver.setDefaultViews(defaultViews);
}
```

컨트롤러 코드는 그대로입니다. 요청 포맷에 따라 스프링이 알아서 적절한 View를 선택합니다.

---

## 8. 에러 처리 — 예외를 메시지와 분리하기

### 대부분 이렇게 씁니다

```java
throw new RuntimeException("사용자를 찾을 수 없습니다.");  // 메시지가 코드에 박힘
```

### 스프링이 준비한 방법 — MessageSourceResolvable

예외 클래스에 "내 메시지 코드가 이거야"라고 알려주고, 실제 메시지는 `.properties` 파일로 분리합니다.

```java
public class SystemException extends RuntimeException implements MessageSourceResolvable {

    @Override
    public String[] getCodes() {
        // 클래스 이름으로 메시지 코드 자동 생성
        // → "Exception.UserNotFoundException"
        return new String[]{"Exception." + getClass().getSimpleName()};
    }
}
```

```properties
# messages_ko.properties
Exception.UserNotFoundException=사용자를 찾을 수 없습니다.
Exception.AccessDeniedException=접근 권한이 없습니다.

# messages_en.properties
Exception.UserNotFoundException=User not found.
Exception.AccessDeniedException=Access denied.
```

`Accept-Language: ko` 요청에는 한국어, `Accept-Language: en` 요청에는 영어 메시지가 자동으로 나갑니다. 코드를 건드리지 않고 메시지만 파일에서 관리합니다.

---

## 9. Feature Toggle — 코드 배포 없이 기능 켜고 끄기

기능 하나를 끄려고 배포하면 위험합니다. 설정값으로 제어하는 패턴입니다.

```yaml
todoapp:
  feature-toggles:
    auth: true              # 인증 기능 on
    online-users-counter: false   # 접속자 수 표시 off
```

```java
@ConfigurationProperties("todoapp.feature-toggles")
public class FeatureTogglesProperties {
    private boolean auth;
    private boolean onlineUsersCounter;
}

@GetMapping("/api/feature-toggles")
public FeatureTogglesProperties featureToggles() {
    return featureTogglesProperties;
}
```

프론트엔드가 앱 시작 시 이 API를 호출해 어떤 기능을 렌더링할지 결정합니다. `application.yml`만 바꾸면 배포 없이 기능을 on/off할 수 있습니다.

---

## 전체 그림 — 요청 하나가 처리되는 흐름

```
HTTP 요청
│
├─ Filter (UserSessionFilter)
│    └─ 세션에서 사용자 꺼내 요청 객체에 담기
│
└─ DispatcherServlet
     │
     ├─ Interceptor.preHandle (RolesVerifyHandlerInterceptor)
     │    └─ @RolesAllowed 확인 → 401/403 또는 통과
     │
     ├─ ArgumentResolver (UserSessionHandlerMethodArgumentResolver)
     │    └─ UserSession 타입 파라미터 자동 주입
     │
     ├─ Controller 메서드 실행
     │
     ├─ ReturnValueHandler (ProfilePictureReturnValueHandler)
     │    └─ ProfilePicture 타입이면 이미지 스트림으로 응답
     │    또는
     └─ ViewResolver (ContentNegotiatingViewResolver)
          └─ Accept 헤더 보고 → HTML / JSON / CSV 중 선택
```

---

## 마무리 — 스프링이 이미 준비해둔 것들

이 교육에서 보여주고자 했던 건 하나입니다.

> **스프링은 이미 확장 포인트를 만들어두었습니다. 우리가 할 일은 그 포인트를 찾아 끼워 넣는 것입니다.**

| 문제 상황 | 스프링 확장 포인트 |
|---|---|
| 모든 요청에 로그인 사용자를 넣어야 함 | `HandlerMethodArgumentResolver` |
| 특정 API에 권한 체크가 필요함 | `HandlerInterceptor` + `@RolesAllowed` |
| 모든 뷰에 공통 데이터가 필요함 | `@ControllerAdvice` + `@ModelAttribute` |
| 응답 포맷을 여러 개 지원해야 함 | `ContentNegotiatingViewResolver` + `AbstractView` |
| 특수한 리턴 타입을 처리해야 함 | `HandlerMethodReturnValueHandler` |
| 설정값을 코드와 분리하고 싶음 | `@ConfigurationProperties` |
| 에러 메시지를 다국어로 지원해야 함 | `MessageSourceResolvable` + `messages.properties` |

이 확장 포인트들을 알고 있으면, 복잡해 보이는 요구사항도 "어디에 코드를 끼워 넣으면 되는지"가 보입니다.
