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

1. `NoUniqueBeanDefinitionException`은 언제 던져지는가?
- 컨텍스트 초기화 중(생성자 주입 대상 빈을 만드는 시점)에 던져진다. 최상위 예외는 `UnsatisfiedDependencyException`이고 이 예외가 root cause로 감싸진다 — 두 겹으로 포장되는 것 자체가 컨테이너 관점 실패와 실제 원인이 별개 레이어에서 감지된다는 증거다.

2. `@PostConstruct`는 어느 시점에 호출되며 직접 `new`는 왜 호출 안 되는가?
- 의존성 주입이 끝난 직후 호출된다. 직접 `new`는 이 콜백 실행 단계 자체를 컨테이너가 관리하지 않으므로 호출되지 않는다.

3. `@PreDestroy`는 `@PostConstruct`와 검증 방식이 왜 다른가?
- `@SpringBootTest` 컨텍스트는 테스트 중 닫히지 않아 관찰 시점이 없다. `AnnotationConfigApplicationContext`를 직접 열고 닫아야 `close()` 시점의 콜백을 확인할 수 있다.

4. 프록시를 거치지 않은 호출에서 advice가 안 걸리는 것을 어떻게 증명했는가?
- `isAopProxy` 확인만으로는 부족하다. 컨테이너 빈(프록시 경유)과 직접 생성 객체(프록시 미경유)를 같은 메서드로 호출해 카운터 증분 여부를 대비시켜야 "프록시 경유 시에만" advice가 동작함이 증명된다.

5. 가장 먼저 깨질 가능성이 높은 부분은?
- self-invocation(`this.findById(...)` 내부 호출) 시 프록시를 우회해 advice가 조용히 빠진다. 지금은 그런 호출이 없어 드러나지 않지만 막는 테스트는 아직 없다.

## 리뷰 반영

자동 리뷰 수신 전
