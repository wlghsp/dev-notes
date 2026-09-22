# Week 2 — 프록시·빈·DI로 보는 스프링 컨테이너 (prep-questions)

대상 저장소: challenge-spring-boot-2026-09-wlghsp-r20
브랜치: submit/week-02__weekly-pr (홈페이지에서 생성)

미션 요구사항 원문은 missions/README.md의 "Week 2" 섹션 참고. 정답을 외워 쓰지 말고, Week 1에서 만든 코드(`todo/TodoRepository.java`, `todo/JdbcTodoRepository.java`, `todo/TodoService.java`)를 직접 열어서 확인한 내용으로 채울 것.

## 필수 1 — 같은 인터페이스의 두 구현체 + @Qualifier/@Primary

- `TodoRepository`는 이미 인터페이스이고 `JdbcTodoRepository` 구현체가 있다. 두 번째 구현체는 무엇으로 만들 것인가? (예: 메모리 구현체, 캐시 구현체 등 — Week 1에서 안 쓴 방식)

> 메모리 구현체(`Map<Long, Todo>` 기반)로 간다. JPA는 새 의존성(spring-boot-starter-data-jpa)이 필요하고 Week 3 트랜잭션 개념을 앞당겨 건드리게 돼서 이번 주 범위(구현체 선택 DI)에 안 맞는다고 판단. 메모리 구현체는 추가 의존성 없이 "같은 인터페이스, 다른 구현"만 순수하게 보여줄 수 있다.

- "선택 전 모호성 실패"를 테스트로 보여주려면, `@Qualifier`나 `@Primary` 없이 두 구현체를 동시에 빈으로 등록했을 때 무슨 예외가 나는지 알아야 한다. 정확한 예외 클래스 이름을 찾아봤는가? (힌트: `NoUniqueBeanDefinitionException` — 실제로 스프링 문서나 공식 API를 찾아서 확인했는가, 아니면 추측만 했는가?)

NoUniqueBeanDefinitionException
동일한 타입의 빈이 2개 이상 존재하여 스프링이 어떤 빈을 주입해야 할지 알 수 없을 때 발생하는 오류입니다. 
해결방법
1. @Primary 사용 : 의존성 주입시 @Primary가 붙은 빈이 우선 주입 받는다. 
2. @Qualifier 사용
빈 등록시 @Qualifier로 등록하고, 주입시에도 @Qualifier로 해당 이름을 사용하여 주입 받는다. 

- 이 예외가 언제 던져지는가? 애플리케이션이 뜨는 시점(컨테이너 초기화 중)인가, 아니면 그 빈을 실제로 주입받으려는 시점인가 — 둘의 차이를 설명할 수 있는가?

기본적으로 애플리에킹션 뜨는 시점(스프링 컨테이너 초기화 중)에 던져집니다.


- `@Qualifier`와 `@Primary` 중 어느 쪽을 쓸 것인가? 각각 언제 더 적합한지 차이를 알고 선택했는가?

1. @Primary를 사용해야 하는 경우 (추천: 메인-서브 구조)
2. @Qualifier를 사용해야 하는 경우 (추천: 수평적 선택 구조)

- `@Primary`가 붙은 구현체와 `@Qualifier`로 명시된 구현체가 동시에 존재하면, 필드/생성자에 `@Qualifier` 없이 그냥 주입받을 때 스프링은 어느 쪽을 선택하는가? 이것도 직접 코드를 짜서 확인할 것인가, 아니면 문서로만 알고 넘어갈 것인가?

스프링은 @Primary가 붙은 구현체를 선택합니다.@Qualifier는 주입받는 지점(필드나 생성자)에 명시적으로 적어주었을 때만 해당 빈을 찾아주는 표식입니다. 따라서 주입받는 곳에 @Qualifier를 생략하면, 스프링은 수많은 후보 중 우선순위를 가진 @Primary 빈을 최종 선택하게 됩니다.

- "격리된 컨텍스트 테스트"라는 표현이 나온다. Week 1의 `JdbcTodoRepositoryTest`가 `@TestConfiguration`으로 빈을 직접 등록했던 패턴을 이번에도 재사용할 수 있는가? "격리"라는 단어가 붙은 이유는 무엇일까 — 전체 `@SpringBootTest`로 띄우면 안 되는 이유가 있는가?


@TestConfiguration 재사용 가능 여부
Week 1에서 사용한 @TestConfiguration 패턴을 이번에도 그대로 재사용할 수 있습니다.특정 테스트 클래스 전용으로 필요한 빈을 수동 등록하거나 가짜(Mock) 객체를 대체 주입할 때 동일하게 유효합니다.

"격리"라는 단어가 붙은 이유
- 상태 공유 방지: 하나의 거대한 통합 테스트 환경을 공유하면, 한 테스트에서 바꾼 데이터나 설정이 다른 테스트에 영향을 줍니다.
- 독립성 보장: 테스트 간 순서나 실행 여부와 상관없이 항상 일정한 결과가 나오도록 컨텍스트를 격리합니다.

전체 @SpringBootTest로 띄우면 안 되는 이유
- 느린 속도: 모든 빈을 다 올리면 스프링 부트 구동 시간이 길어져 피드백 루프가 느려집니다.
- 불필요한 의존성: 리포지토리 레이어만 테스트하고 싶어도 웹(Web), 보안(Security), 서비스(Service) 등 관련 없는 빈까지 모두 로드되어 자원이 낭비됩니다.
- 사이드 이펙트: 전역 설정이나 다른 컴포넌트의 초기화 로직이 얽혀 원치 않는 테스트 실패가 발생할 수 있습니다.

## 필수 2 — 생성자 주입 vs 직접 new — 관찰 가능한 차이

- `TodoService`는 이미 생성자 주입을 쓰고 있다. 이 서비스를 스프링 컨테이너 없이 `new TodoService(new JdbcTodoRepository(...))`로 직접 만들면 어떤 점에서 차이가 나는가? (의존성 교체 용이성, 생명주기 콜백 적용 여부, AOP 부가기능 적용 여부 중 어느 것을 보여줄 것인가?)

1. AOP 부가 기능 적용 여부 
직접 new로 생성 시 @Transactional, @Secured 로깅 등 AOP 기반의 부가기능이 전혀 동작하지 않습니다.
이유: 스프링은 컨테이너에 빈을 등록할 때 원래 클래스를 그대로 등록하는 것이 아니라, 프록시 객체를 중간에 생성하여 감싸는 방식으로 AOP를 구현합니다. 개발자가 new 로 직접 객체를 만들면 프록시가 개입할 틈이 없으므로 순수한 비즈니스 로직만 실행되고 트랜잭션 시작/커밋/롤백 등의 부가기능은 누락됩니다.

2. 생명주기 콜백 적용 여부
직접 new로 생성 시: @PostConstruct, @PreDestroy나 InitializingBean 같은 스프링의 초기화 및 종료 메서드가 자동으로 호출되지 않습니다.
이유: 생명주기 콜백은 스프링 컨테이너가 빈의 생성, 의존성 주입을 마친 후 특정 시점에 메서드를 대신 호출해 주는 기능입니다. 컨테이너의 관리를 벗어나면 개발자가 직접 초기화 메서드를 호출(service.init())하고, 종료 시점에 파괴 메서드를 직접 챙겨야 합니다.

- "의존성, 생명주기 또는 부가기능 중 하나"라고 했다 — 셋 중 어느 걸 테스트로 보여줄지 미리 정했는가? (3번 AOP 요구사항과 겹치지 않게 고르는 게 좋을 수 있다)

> 생명주기
1. @PostConstruct 콜백은 컨테이너가 빈을 만들 때만 자동 호출됨
2. new로 만들면 이 콜백이 아예 실행되지 않음
3. 로그 한 줄이나 boolean 플래그로 호출됐는지 안됐는지 딱 떨어지게 assert 가능

- 셋 중 "의존성"을 고른다면, 컨테이너가 관리하는 빈은 인터페이스(`TodoRepository`)에 의존하고 직접 만든 객체는 구체 클래스(`JdbcTodoRepository`)에 의존하게 되는 상황을 어떻게 코드로 대비시킬 것인가? 아니면 "부가기능"을 골라서 3번(AOP)과 자연스럽게 이어지게 할 것인가 — 어느 쪽이 관찰 가능한 차이를 더 명확하게 보여준다고 판단했는가?

> 해당 없음 — 생명주기로 확정. 의존성은 `new TodoService(...)`로 만들어도 여전히 `TodoRepository` 인터페이스 타입을 받는 구조라 "인터페이스 vs 구체 클래스 의존" 차이가 코드상 뚜렷하게 안 드러나서 제외. 부가기능은 3번(AOP) 요구사항과 내용이 겹쳐서 제외.

- 직접 `new`한 객체와 컨테이너 빈이 "다르다"는 걸 테스트에서 어떻게 단언(assert)할 것인가? 단순히 "코드가 다르게 생겼다"가 아니라 실행 시점에 관찰 가능한 차이(예: 프록시 여부, 예외 발생 여부)로 보여줘야 한다는 걸 염두에 뒀는가?

> 스프링의 AOP 프록시 적용 여부와 의존성 주입(DI) 상태를 통해 실행 시점에 명확하게 단언 할 수 있습니다. 


- 컨테이너가 빈을 만들 때 내부적으로 어떤 단계를 거치는지 알고 있는가? (인스턴스 생성 → 의존성 주입 → `@PostConstruct` 등 초기화 콜백 → 사용 → `@PreDestroy` 등 소멸 콜백) 직접 `new`한 객체는 이 단계 중 정확히 어디부터 어디까지를 "받지 못하는가"?

### 스프링 빈의 전체 생명주기 단계
1. 인스턴스 생성: 스프링이 설정 정보(AppConfig, @Component 등)를 읽어 빈 객체를 생성합니다. 
2. 의존 관계 주입(DI): @Autowired나 생성자 주입을 통해 필요한 의존성을 주입합니다. 
3. 초기화 콜백: 
- BeanNameAware, BeanFactoryAware 등 각종 Aware 인터페이스 적용
- @PostConstruct 어노테이션이 붙은 메서드 실행
- InitializingBean 인터페이스의 afterProperitesSet() 실행
- 설정 정보에 지정한 initMethod 실행
4. 사용: 애플리케이션 내에서 빈을 가져와 비즈니스 로직을 수행합니다. 
5. 소멸 콜백: (컨테이너가 종료될 때)
- @PreDestroy 어노테이션이 붙은 메서드 실행
- DisposableBean 인터페이스의 destroy() 실행
- 설정 정보에 지정한 destroyMethod 실행 

- "생명주기"를 고른다면, `@PostConstruct`가 붙은 메서드가 언제 호출되는지 — 생성자 실행 직후인가, 의존성 주입이 끝난 직후인가? 이 순서를 착각하면 어떤 버그가 생길 수 있는가? 직접 `new`했을 때는 이 콜백이 아예 호출되지 않는다는 걸 로그나 assert로 어떻게 보여줄 것인가?

> 의존성 주입이 끝난 직후에 호출됩니다. 

- 생성자 실행 직후: 생성자가 실행될때는 아직 필드 주입이나 @Autowired 주입이 완료되지 않아 주입받을 객체가 null 상태입니다.
- 의존성 주입이 끝난 직후: 예. 스프링 컨테이너가 빈을 생성하고 필요한 의존성을 모두 주입한 바로 그 시점에 @PostConstruct 메서드가 실행됩니다. 따라서 이 메서드 안에서는 주입된 객체들을 안전하게 사용할 수 있습니다. 


## 필수 3 — AOP 프록시 검증

- Todo 서비스의 어떤 메서드에 부가기능을 적용할 것인가? (예: 실행 시간 로깅, 호출 횟수 기록 등)

> `TodoService.findById`에 호출 횟수 기록 advice를 적용한다. `create`는 쓰기 작업이라 `save`/`assignId` 흐름과 advice 순서가 겹쳐서 검증이 복잡해질 수 있고, 실행 시간 로깅은 값이 매번 달라져 assert하기 애매하다. `findById`는 조회라 부작용이 없고, 호출 횟수는 정수로 딱 떨어지게 검증 가능 — Week 1 `ProbeAspect`(`ProxyProbe.call()`)와 같은 패턴.

- `@Aspect`를 어떻게 등록할 것인가? Week 1의 `ChallengeApplicationTest`에 있던 `ProbeAspect` 예제(`@Around`, `bean(...)` 포인트컷)를 참고했는가?

> Week 1의 `ProbeAspect`는 `@TestConfiguration` 내부 static 클래스로 테스트에서만 수동 등록됐지만, 이번엔 운영 코드(`co.dingcodingco.challenge.todo` 패키지)에 `@Aspect @Component`로 등록한다. 미션 문구가 "AOP 부가기능을 적용하고"라 실험이 아니라 실제 컴포넌트 스캔으로 항상 동작하는 게 요구사항에 더 맞다고 판단. `@Around` + 포인트컷 구조는 `ProbeAspect` 패턴을 참고한다.

- `AopUtils.isAopProxy(...)`와 `AopUtils.getTargetClass(...)`를 테스트에서 어떻게 쓸 것인지 확인했는가?

> Week 1 `ChallengeApplicationTest`의 `ProxyProbe` 검증 패턴을 그대로 따른다. 컨테이너에서 꺼낸 `TodoService` 빈으로 `AopUtils.isAopProxy(todoService)`가 `true`인지 확인해 프록시로 감싸졌음을 증명하고, `AopUtils.getTargetClass(todoService)`가 `TodoService.class`인지 확인해 프록시 뒤에 숨은 원본 클래스를 검증한다. CGLIB 프록시라 `todoService.getClass()`는 `TodoService$$SpringCGLIB$$...` 같은 이름이 나오지만, `getTargetClass()`는 그 원본을 알려준다는 차이를 이 테스트로 같이 보여줄 수 있다.

- "프록시를 통한 호출에서만" 이라는 표현이 있다 — 프록시를 거치지 않고 대상 객체를 직접 호출하면 advice가 안 걸린다는 걸 어떻게 대비 테스트로 보여줄 것인가?

> 필수 2번에서 준비한 "직접 new한 TodoService"를 재사용한다. 컨테이너에서 꺼낸 프록시 `todoService.findById(...)` 호출 후 카운터가 1 증가하는 것과, `new TodoService(new JdbcTodoRepository(...))`로 직접 만든 원본에서 같은 메서드를 호출해도 카운터가 그대로인 것을 한 테스트 안에서 대비시킨다. 카운터는 advice 안에서만 증가시키고 `findById` 메서드 자체에는 카운트 로직을 넣지 않아야, 증가 여부가 순수하게 "프록시 경유 여부"의 증거가 된다.

- 포인트컷을 `bean(빈이름)`으로 지정할지 `execution(...)` 표현식으로 지정할지 — Week 1 `ProbeAspect`는 `bean(proxyProbe)`를 썼다. 이번에도 같은 방식을 쓸 것인가, 아니면 메서드 시그니처 기반(`execution`)으로 더 세밀하게 지정할 이유가 있는가?

> 메서드 시그니처 기반(`execution`)을 쓴다. `bean(proxyProbe)`는 빈 이름 하나에 전체가 걸리는 방식이라, `TodoService` 안의 `create`와 `findById` 중 `findById`에만 advice를 걸고 싶은 이번 요구사항엔 세밀함이 부족하다. `execution(* co.dingcodingco.challenge.todo.TodoService.findById(..))`처럼 메서드 시그니처로 지정하면 같은 클래스의 다른 메서드(`create`)는 advice 대상에서 자연스럽게 제외된다.

- "advice와 대상 호출이 기대한 순서와 횟수로 기록되는지"를 검증하려면 Week 1의 `ProbeEvents`처럼 호출 순서를 기록하는 보조 객체가 필요하다. 이번에도 비슷한 구조(리스트에 이벤트 문자열 쌓기)를 재사용할 것인가?

> 네, 이벤트 문자열을 리스트에 쌓는 보조 객체(ProbeEvents 유사 구조)를 재사용하는 것이 호출 순서와 횟수를 검증하는 가장 직관적이고 효과적인 방법입니다.

- `TodoService`에 이미 생성자 주입된 `TodoRepository`가 있는 상태에서 AOP 프록시까지 씌우면, 스프링이 실제로 감싸는 대상이 `TodoService` 원본 객체인지 아니면 그 프록시인지 — 이 둘의 관계를 정확히 설명할 수 있는가?

> 컨테이너가 감싸는 건 `TodoService` 원본 객체다. 스프링은 먼저 진짜 `TodoService`(생성자로 `TodoRepository`를 주입받은 상태)를 만들고, 그 다음 그걸 프록시로 한 번 더 감싼다. 최종적으로 컨트롤러 등 다른 빈이 주입받는 건 이 프록시다. `TodoRepository` 필드 자체는 이번 주 AOP 대상이 아니라서 원본 그대로지만, 이건 "TodoService 안에 있어서"가 아니라 "AOP를 안 걸었기 때문"이다 — 프록시 여부는 각 빈이 AOP 대상인지에 따라 독립적으로 결정된다.

- 스프링 AOP가 프록시를 만드는 두 가지 방식(JDK 동적 프록시 vs CGLIB)을 아는가? `TodoService`가 인터페이스를 구현하지 않은 순수 클래스라면 어느 방식이 선택되는가? `AopUtils.getTargetClass(...)`로 확인했을 때 나오는 클래스명에 `$$SpringCGLIB$$` 같은 패턴이 보이는지 직접 로그로 찍어봤는가?

> CGLIB이 선택된다. `TodoService`는 인터페이스를 구현하지 않은 순수 클래스라서, JDK 동적 프록시(인터페이스 기반)는 적용 불가능하고 스프링이 자동으로 CGLIB(클래스 상속 기반)으로 프록시를 만든다. `AopUtils.getTargetClass(todoService)`가 아니라 `todoService.getClass().getName()`을 로그로 찍어보면 `TodoService$$SpringCGLIB$$...` 형태의 이름이 나오는 것까지 확인했다.

- `@Around` advice 안에서 `joinPoint.proceed()`를 호출하는 시점 전후로 코드를 쓰면 "before"와 "after"가 왜 나뉘는지 — `proceed()`가 정확히 무엇을 하는 호출인지 설명할 수 있는가? `proceed()`를 아예 안 부르면 대상 메서드는 실행되는가, 안 되는가? 이것도 실험해서 확인했는가?

1. joinPoint.proceed()는 정확히 무엇을 하는가?
joinPoint.proceed()는 실제 비즈니스 로직(대상 메서드) 또는 다음 인터셉터(또는 어드바이스)를 호출하는 명령어입니다.스프링 AOP는 내부적으로 프록시(Proxy) 패턴을 사용합니다. 클라이언트가 특정 메서드를 호출하면 실제 객체가 아니라 프록시 객체가 그 요청을 가로챕니다. 이때 프록시 내부에서 @Around 코드가 실행되며, proceed()를 만나는 순간 "내가 가로챘던 원래 메서드를 이제 실행해라" 하고 바톤을 넘기는 역할을 합니다.

2. 전후로 코드를 쓰면 Before와 After가 나뉘는 이유
proceed() 호출은 동기식(Synchronous) 메서드 호출입니다. 따라서 코드가 실행되는 흐름(Thread)이 proceed()를 기점으로 완전히 쪼개집니다.Before 영역: proceed()가 호출되기 전의 코드입니다. 프록시가 요청을 가로챈 직후에 실행되므로, 대상 메서드가 시작되기 전에 처리해야 할 일(로그 출력, 파라미터 조작, 권한 검증 등)을 수행합니다.joinPoint.proceed() 실행: 비즈니스 로직이 실행되고 결과값을 반환하거나 예외를 던집니다.After 영역: proceed()가 제어권을 다시 반환한 후의 코드입니다. 실제 메서드가 성공적으로 끝난 뒤에 실행되므로, 결과값을 가공하거나 실행 시간을 측정하는 등의 사후 처리 작업을 수행합니다.


- 같은 클래스 안의 다른 메서드가 AOP 적용 대상 메서드를 내부적으로 호출하면(self-invocation) advice가 걸리는가, 안 걸리는가? 그 이유는 프록시 구조상 왜 그런가? (Week 3 트랜잭션 프록시 우회 문제와 같은 원리라는 걸 지금 짚어볼 수 있는가?)

> 안 걸린다. 스프링 AOP는 프록시가 외부 호출을 가로채는 방식이라, 클라이언트가 메서드를 호출할 때만(프록시를 거쳐서) advice가 동작한다. 일단 외부에서 프록시를 거쳐 타깃 객체 내부로 진입한 뒤, 그 안에서 `this.targetMethod()` 형태로 다른 메서드를 호출하면 프록시를 다시 거치지 않고 타깃 객체 자신의 메서드를 직접 호출하게 된다. 이게 Week 3의 `@Transactional` 프록시 우회 문제와 정확히 같은 원리 — 같은 클래스 내부 호출이 트랜잭션 프록시도 우회한다.
>
> 해결 방법(구조 분리): 내부 호출이 발생하지 않도록 해당 메서드를 별도의 서비스 클래스로 분리해 외부 호출 형태로 바꾼다.


## 다음 단계

이 파일에 답을 채운 뒤 execution-guide.md를 만들어 실제 구현 순서를 정리한다. 4번(선택 확장, 빈 생명주기 콜백/스코프 프록시)은 필수 1~3이 끝난 뒤에 판단한다.
