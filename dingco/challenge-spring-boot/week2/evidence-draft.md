## 변경

`TodoRepository`에 두 번째 구현체 `InMemoryTodoRepository`를 추가하고 `JdbcTodoRepository`에 `@Primary`를 붙여 모호성을 해소했다.
`TodoService`에 `@PostConstruct`/`@PreDestroy`를 추가해 컨테이너 빈과 직접 `new`한 객체의 생명주기 차이를 관찰 가능하게 만들었다.
`findById`에만 걸리는 `CallCountingAspect`(`execution(...)` 포인트컷)를 운영 코드에 등록해 AOP 프록시 적용 여부를 검증했다.

## 검증

```
JAVA_HOME=<temurin-17 경로> mvn --batch-mode --no-transfer-progress test
```

Tests run: 25, Failures: 0 — BUILD SUCCESS

- 모호성: `@Primary` 없이 두 구현체 등록 시 `UnsatisfiedDependencyException`(root cause `NoUniqueBeanDefinitionException`), `@Primary` 적용 후 `JdbcTodoRepository` 선택 확인
- 생명주기: 컨테이너 빈은 `@PostConstruct`/`@PreDestroy`가 호출됨(`isInitialized`/`isDestroyed` true), 직접 `new`한 객체는 둘 다 false
- AOP: `AopUtils.isAopProxy`로 프록시 확인, 프록시 경유 호출은 카운터 증가, 직접 `new`한 객체 호출은 카운터 미증가
- Week 1 기존 테스트 전부 회귀 없음

참고: `JAVA_HOME` 미지정 시 시스템 기본 JDK(26 preview)로 Mockito/ByteBuddy 계측이 실패했다. Temurin 17로 명시하니 통과 — pom.xml의 `<java.version>17</java.version>`과 실행 JDK가 일치해야 한다.

## 선택 근거

- 두 번째 구현체는 JPA 대신 메모리 구현체 — 새 의존성·Week 3 범위를 앞당기지 않기 위함
- 모호성 해소는 `@Qualifier` 대신 `@Primary` — 메인(JDBC)/서브(메모리) 구조에 기본 우선순위가 더 자연스러움
- 필수 2번 관찰 포인트로 "생명주기" 선택 — "의존성"은 직접 new에서도 차이가 안 드러났고 "부가기능"은 필수 3번과 겹쳐서 제외
- AOP 대상은 `findById` — 조회라 부작용 없고 호출 횟수로 딱 떨어지게 검증 가능
- 포인트컷은 `bean(...)` 대신 `execution(...)` — `create`는 제외하고 `findById`만 정밀 타게팅
- `callCounter.increment()`는 `finally`에 배치 — 예외 발생 시에도 호출 사실이 누락되지 않도록
- 미확인 한계: self-invocation 시 advice 우회 상황은 원리만 정리했고 테스트로 재현하지 않음

## 근거형 질문

1. 스프링이 주입할 구현체를 결정할 수 없을 때 어떤 일이 발생하며 어떻게 해결했나요?
- 최초 판단: 후보가 여럿이면 컨테이너가 뜨는 시점에 실패할 것이라 예상했다.
- 근거: `TodoRepositoryAmbiguityTest`에서 `TodoRepository` 구현체 2개를 등록하고 컨텍스트를 띄워 재현했다.
- 검증 후 답변: `NoUniqueBeanDefinitionException`(root cause, 최상위는 `UnsatisfiedDependencyException`)이 발생했다. `JdbcTodoRepository`에 `@Primary`를 붙여 해결했다.

2. 생성자 주입이 필드 주입보다 테스트와 불변성에 유리한 이유는 무엇인가요?
- 최초 판단: 생성자 주입이면 컨테이너 없이도 원하는 구현체를 직접 넣어 테스트할 수 있을 거라 판단했다.
- 근거: `TodoServiceAopTest`에서 `new TodoService(new InMemoryTodoRepository())`로 직접 인스턴스를 만들어 테스트했다.
- 검증 후 답변: 실제로 가능했다. 필드도 `final`로 선언돼 있어 생성 이후 값이 바뀌지 않는다는 게 보장된다 — 필드 주입은 이 둘 다 불가능하다.

3. 컨테이너가 관리하는 객체와 직접 생성한 객체의 가장 중요한 차이는 무엇인가요?
- 최초 판단: 생명주기 콜백(`@PostConstruct`, `@PreDestroy`) 실행 여부가 가장 뚜렷한 차이일 것이라 판단했다.
- 근거: `TodoServiceLifecycleTest`, `TodoServiceDestroyLifecycleTest`에서 컨테이너 빈과 직접 `new`한 객체의 `isInitialized()`/`isDestroyed()`를 비교했다.
- 검증 후 답변: 컨테이너 빈은 둘 다 true, 직접 `new`한 객체는 둘 다 false로 확인됐다.

4. 프록시 객체인지 원본 객체인지 코드로 어떻게 확인할 수 있나요?
- 최초 판단: `AopUtils`로 정적 확인이 가능할 거라 판단했다.
- 근거: `TodoServiceAopTest`에서 `AopUtils.isAopProxy(bean)`, `AopUtils.getTargetClass(bean)`을 확인하고, 프록시/직접 생성 객체 각각 `findById` 호출 후 `CallCounter` 증가 여부도 비교했다.
- 검증 후 답변: `isAopProxy`가 true, `getTargetClass`가 원본 클래스를 반환했다. 실행 결과 대비(카운터 증가 여부)까지 더해야 "실제로 advice가 동작하는지"까지 확인된다.

5. 이번 실험의 호출 순서 로그가 없었다면 어떤 설명을 검증할 수 없었나요?
- 최초 판단: 호출 횟수만으로는 advice가 대상 메서드를 앞뒤로 "감싸는" 구조인지까지는 못 볼 것이라 판단했다.
- 근거: `CallCounter`는 횟수만 세고, Week 1 `ProbeEvents`처럼 문자열 순서를 기록하지 않는다.
- 검증 후 답변: `@Around`의 before/after 동작(advice가 대상 메서드를 감싸는 구조)은 순서 로그 없이는 검증할 수 없다 — 이번 구현은 횟수만 확인했다.

## 리뷰 반영

자동 리뷰 수신 전
