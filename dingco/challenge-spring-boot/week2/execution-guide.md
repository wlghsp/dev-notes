# Week 2 — 실행 가이드

prep-questions.md 답변 기준으로 실제 구현 순서를 정리한다. Week 1과 같은 방식으로 진행한다 — 실패하는 테스트(Red)를 먼저 준비하고, 통과시키는 최소 구현은 직접 작성해서 Green으로 만들 것.

대상 저장소: challenge-spring-boot-2026-09-wlghsp-r20
브랜치: `submit/week-02__weekly-pr` (홈페이지에서 생성 후 체크아웃)

## 전체 순서

prep-questions에서 이미 정한 것들:

- 두 번째 `TodoRepository` 구현체 = 메모리 구현체(`Map<Long, Todo>`)
- 필수 2(생성자 주입 vs 직접 new) = 생명주기(`@PostConstruct`)로 증명
- 필수 3(AOP 대상) = `TodoService.findById` + 호출 횟수 기록, 운영 코드에 `@Aspect @Component`로 등록, 포인트컷은 `execution(...)`

1. 메모리 구현체 `InMemoryTodoRepository` 작성
2. 모호성 실패 테스트 → `@Primary`/`@Qualifier`로 해결
3. `TodoService`에 `@PostConstruct` 추가 → 생성자 주입 빈 vs 직접 `new` 비교 테스트
4. `CallCountingAspect` 작성 → `AopUtils` + 호출 순서/횟수 검증
5. 실행 검증 → evidence 기록

패키지는 기존 코드와 맞춰 `co.dingcodingco.challenge.todo` 하위에 구성한다.

---

## 1. 두 번째 저장소 구현체 — InMemoryTodoRepository

### Red — 테스트부터 작성

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;

class InMemoryTodoRepositoryTest {

    private final TodoRepository repository = new InMemoryTodoRepository();

    @Test
    void savedTodoCanBeFoundById() {
        Todo saved = repository.save(new Todo("메모리 저장", false));

        Todo found = repository.findById(saved.getId()).orElseThrow();

        assertThat(found.getTitle()).isEqualTo("메모리 저장");
    }

    @Test
    void returnsEmptyWhenNotFound() {
        assertThat(repository.findById(999999L)).isEmpty();
    }
}
```

### Green — 직접 작성할 것

`InMemoryTodoRepository`가 `TodoRepository`를 구현한다. `JdbcTodoRepository`의 `save`가 `GeneratedKeyHolder`로 id를 채우듯, 여기서는 `AtomicLong` 같은 걸로 순번을 매기고 `assignId(...)`로 채운다. `Map<Long, Todo>`에 담아두면 된다.

이 시점에서 `TodoRepository` 구현체가 두 개(`JdbcTodoRepository`, `InMemoryTodoRepository`)가 됐다. 둘 다 `@Repository`를 붙이면 다음 단계에서 모호성 문제가 바로 재현된다.

## 2. 모호성 실패와 명시적 선택

### Red — 모호성 실패를 먼저 테스트로 확인

prep-questions에서 조사한 `NoUniqueBeanDefinitionException`을 실제로 발생시켜본다. `@TestConfiguration`으로 두 구현체를 `@Qualifier`/`@Primary` 없이 동시에 등록하고 `TodoService`를 주입받으려 하면 컨텍스트 로딩 자체가 실패한다.

실제로 돌려보면 스프링이 던지는 최상위 예외는 `UnsatisfiedDependencyException`(생성자 주입 실패)이고, `NoUniqueBeanDefinitionException`(후보가 여럿이라 못 고르겠다는 진짜 원인)은 그 안에 root cause로 감싸져 있다. 예외가 두 겹으로 포장된다는 것 자체가, 컨테이너 초기화 중 어느 지점에서 실패가 감지되는지를 보여주는 증거이기도 하다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.Mockito.mock;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.beans.factory.UnsatisfiedDependencyException;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.core.JdbcTemplate;

class TodoRepositoryAmbiguityTest {

    @Test
    void ambiguousRepositoriesFailContainerStartup() {
        assertThatThrownBy(() -> {
            try (ConfigurableApplicationContext context = new AnnotationConfigApplicationContext(AmbiguousConfig.class)) {
                context.getBean(TodoService.class);
            }
        })
            .isInstanceOf(UnsatisfiedDependencyException.class)
            .hasRootCauseInstanceOf(NoUniqueBeanDefinitionException.class);
    }

    @TestConfiguration
    static class AmbiguousConfig {
        @Bean
        TodoRepository jdbcTodoRepository(JdbcTemplate jdbcTemplate) {
            return new JdbcTodoRepository(jdbcTemplate);
        }

        @Bean
        TodoRepository inMemoryTodoRepository() {
            return new InMemoryTodoRepository();
        }

        @Bean
        JdbcTemplate jdbcTemplate() {
            // 이 테스트는 컨테이너 초기화 실패만 보면 되므로 실제 DataSource는 불필요.
            // JdbcTodoRepository 생성자를 채우기 위한 용도로만 mock을 쓴다.
            return mock(JdbcTemplate.class);
        }

        @Bean
        TodoService todoService(TodoRepository todoRepository) {
            return new TodoService(todoRepository);
        }
    }
}
```

`jdbcTemplate()`이 `Mockito.mock(JdbcTemplate.class)`을 반환하는 이유: 이 테스트는 JDBC가 실제로 동작하는지가 아니라 빈 등록 자체가 실패하는지만 보면 되므로, `JdbcTodoRepository` 생성자를 채울 용도로만 가벼운 mock을 쓴다. (`null`을 반환하면 안 된다 — `@Bean` 메서드가 `null`을 반환하면 스프링은 그 타입의 빈이 아예 없다고 취급해서, 의도한 모호성 예외 대신 "그런 빈이 없다"는 `UnsatisfiedDependencyException`이 더 앞단에서 먼저 터진다.)

### Green — @Primary로 모호성 해소

prep-questions 답변대로 `@Primary`를 고른 이유를 실제 코드로 확인한다. `JdbcTodoRepository`에 `@Primary`를 붙이면 위 테스트의 모호성 실패가 사라지고, `TodoService`는 자동으로 `JdbcTodoRepository`를 받는다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.jdbc.core.JdbcTemplate;

@JdbcTest
@Import(TodoRepositorySelectionTest.RepositoryConfig.class)
class TodoRepositorySelectionTest {

    @Autowired
    private TodoRepository todoRepository;

    @Test
    void primaryRepositoryIsSelectedWithoutAmbiguity() {
        assertThat(todoRepository).isInstanceOf(JdbcTodoRepository.class);
    }

    @TestConfiguration
    static class RepositoryConfig {
        @Bean
        @org.springframework.context.annotation.Primary
        TodoRepository jdbcTodoRepository(JdbcTemplate jdbcTemplate) {
            return new JdbcTodoRepository(jdbcTemplate);
        }

        @Bean
        TodoRepository inMemoryTodoRepository() {
            return new InMemoryTodoRepository();
        }
    }
}
```

실제 운영 코드(`JdbcTodoRepository`, `InMemoryTodoRepository` 클래스 선언부)에 `@Primary`를 어디에 붙일지는 직접 정할 것 — 클래스 레벨 `@Primary` vs `@Bean` 메서드 레벨 `@Primary` 중 이 프로젝트 구조(컴포넌트 스캔 기반 `@Repository`)에 맞는 쪽을 고른다.

## 2-보완. 빈 생명주기 콜백 — 컨테이너 빈 vs 직접 new

### Red — 테스트부터 작성

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class TodoServiceLifecycleTest {

    @Autowired
    private TodoService containerManagedService;

    @Autowired
    private TodoRepository todoRepository;

    @Test
    void containerBeanRunsPostConstruct() {
        assertThat(containerManagedService.isInitialized()).isTrue();
    }

    @Test
    void directlyCreatedObjectSkipsPostConstruct() {
        TodoService rawService = new TodoService(todoRepository);

        assertThat(rawService.isInitialized()).isFalse();
    }
}
```

### Green — 직접 작성할 것

`TodoService`에 `@PostConstruct`가 붙은 메서드를 추가한다. prep-questions에서 이미 "의존성 주입이 끝난 직후 호출된다"고 정리했으니, 그 메서드 안에서 `initialized` 필드를 `true`로 바꾼다. `isInitialized()` getter도 추가한다.

이 필드는 순수하게 "컨테이너가 이 콜백을 호출해줬는지" 표시하는 용도다 — 실제 초기화 로직(DB 연결 준비 등)이 필요한 건 아니므로, 비즈니스 의미는 없고 관찰용 플래그라는 걸 염두에 둘 것.

## 3. AOP — findById 호출 횟수 기록

### Red — 테스트부터 작성

Week 1의 `ProxyProbe`/`ProbeEvents`/`ProbeAspect` 3종 세트와 같은 구조를 쓴다. 다만 이번엔 운영 코드에 실제로 등록하므로, 이 테스트는 `@SpringBootTest`로 실제 컨텍스트에서 검증한다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.aop.support.AopUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class TodoServiceAopTest {

    @Autowired
    private TodoService todoService;

    @Autowired
    private CallCounter callCounter;

    @Test
    void todoServiceIsWrappedByProxy() {
        assertThat(AopUtils.isAopProxy(todoService)).isTrue();
        assertThat(AopUtils.getTargetClass(todoService)).isEqualTo(TodoService.class);
    }

    @Test
    void findByIdThroughProxyIncrementsCallCount() {
        callCounter.reset();
        Todo saved = todoService.create("AOP 확인용", false);

        todoService.findById(saved.getId());

        assertThat(callCounter.getCount()).isEqualTo(1);
    }

    @Test
    void directlyCreatedServiceBypassesAdvice() {
        callCounter.reset();
        TodoService rawService = new TodoService(new InMemoryTodoRepository());
        Todo saved = rawService.create("우회 확인용", false);

        rawService.findById(saved.getId());

        assertThat(callCounter.getCount()).isEqualTo(0);
    }
}
```

### Green — 직접 작성할 것

세 가지가 필요하다. `CallCounter`는 순수 보조 클래스라 바로 제공한다 — 직접 만들어야 하는 건 `CallCountingAspect`다.

**`CallCounter`** — 호출 횟수를 세는 보조 빈. `@Component`로 등록해서 테스트와 Aspect가 같은 인스턴스를 공유한다.

```java
package co.dingcodingco.challenge.todo;

import java.util.concurrent.atomic.AtomicInteger;
import org.springframework.stereotype.Component;

@Component
public class CallCounter {

    private final AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }

    public void reset() {
        count.set(0);
    }
}
```

**`CallCountingAspect`** — `@Aspect @Component`로 등록하고, `CallCounter`를 생성자로 주입받는다. 포인트컷은 `execution(...)`으로 `TodoService.findById`만 정확히 지정한다 — Week 1 `ProbeAspect`가 `@Around("bean(proxyProbe)")`를 쓴 것과 달리, 이번엔 메서드 시그니처 기반이다.

```java
package co.dingcodingco.challenge.todo;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class CallCountingAspect {

    private final CallCounter callCounter;

    public CallCountingAspect(CallCounter callCounter) {
        this.callCounter = callCounter;
    }

    @Around("execution(* co.dingcodingco.challenge.todo.TodoService.findById(..))")
    public Object countCall(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            return joinPoint.proceed();
        } finally {
            callCounter.increment();
        }
    }
}
```

`callCounter.increment()`를 `finally` 블록에 둔 이유: `findById`가 `TodoNotFoundException`을 던지는 경우(존재하지 않는 id 조회)에도 "호출은 일어났다"는 사실 자체는 기록돼야 하기 때문이다. `proceed()` 다음 줄에만 두면 예외가 발생했을 때 카운트가 누락된다.

`directlyCreatedServiceBypassesAdvice` 테스트에서 `new TodoService(new InMemoryTodoRepository())`를 쓰는 이유: `JdbcTodoRepository`는 `JdbcTemplate`이 필요해서 직접 `new`하기 번거롭지만, `InMemoryTodoRepository`는 인자 없이 바로 만들 수 있어 이 테스트를 가볍게 유지한다. 1번에서 만든 메모리 구현체가 여기서 재사용된다.

## 4. 선택 확장 — @PreDestroy 소멸 콜백

필수 2번(`@PostConstruct`, 초기화 시점)과 짝을 이루는 종료 시점 콜백을 추가한다. "컨테이너가 관리하는 경계를 한 가지 더 비교"하라는 선택 확장 요구를, 이미 만든 생명주기 테스트 구조를 그대로 재사용해서 채운다.

### Red — 테스트부터 작성

`@PreDestroy`는 컨테이너가 **종료될 때** 호출되므로, 컨텍스트를 살아있는 채로 두고 관찰할 수 없다. 별도의 `AnnotationConfigApplicationContext`를 직접 띄우고 닫아서, "닫히는 순간" 콜백이 불렸는지 확인해야 한다.

```java
package co.dingcodingco.challenge.todo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

class TodoServiceDestroyLifecycleTest {

    @Test
    void containerBeanRunsPreDestroyOnContextClose() {
        TodoService service;
        try (AnnotationConfigApplicationContext context =
                 new AnnotationConfigApplicationContext(SingleRepositoryConfig.class)) {
            service = context.getBean(TodoService.class);
            assertThat(service.isDestroyed()).isFalse();
        }
        // try 블록을 벗어나며 context.close()가 호출된 뒤
        assertThat(service.isDestroyed()).isTrue();
    }

    @Test
    void directlyCreatedObjectNeverRunsPreDestroy() {
        TodoService rawService = new TodoService(new InMemoryTodoRepository());

        // 컨테이너를 거치지 않았으니 어떤 시점에도 호출될 일이 없다
        assertThat(rawService.isDestroyed()).isFalse();
    }

    @Configuration
    static class SingleRepositoryConfig {
        @Bean
        TodoRepository todoRepository() {
            return new InMemoryTodoRepository();
        }

        @Bean
        TodoService todoService(TodoRepository todoRepository) {
            return new TodoService(todoRepository);
        }
    }
}
```

이 테스트에서 `@Primary`/`@Qualifier` 없이 `TodoRepository` 빈을 하나만 등록한 이유: 여기서 검증하려는 건 모호성이 아니라 생명주기이므로, 필수 1번 문제와 섞이지 않도록 후보를 하나로 좁혔다.

### Green — 직접 작성할 것

`TodoService`에 `@PreDestroy`가 붙은 메서드를 추가한다. `@PostConstruct`로 이미 만든 `initialized` 필드와 같은 패턴으로 `destroyed` 필드와 `isDestroyed()`를 만들면 된다.

컨테이너가 이 콜백을 호출하는 시점은 `ApplicationContext`가 닫힐 때(`close()` 호출 또는 JVM 종료 훅)다. `@SpringBootTest`로 띄운 컨텍스트는 테스트 클래스 실행 중엔 안 닫히므로, 이 콜백을 직접 관찰하려면 위 테스트처럼 `AnnotationConfigApplicationContext`를 스스로 열고 닫아야 한다 — `@PostConstruct`를 `@SpringBootTest`로 편하게 검증했던 것과 달리, `@PreDestroy`는 컨텍스트 생명주기를 직접 다뤄야 관찰 가능하다는 차이를 여기서 확인하게 된다.

## 다음 단계

1. 네 파일(모호성/생명주기/AOP proxy/AOP count) 테스트가 모두 통과하는지 확인
2. evidence-draft.md 작성 (변경/검증/선택 근거/근거형 질문/리뷰 반영)
3. evidence-draft를 `evidence/week-02__weekly-pr.md`로 옮김
4. PR 생성 → 자동 검사·AI 리뷰 확인 → review-notes.md 기록
