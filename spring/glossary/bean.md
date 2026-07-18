# 빈 (Bean)과 스프링 IoC 컨테이너

스프링 IoC 컨테이너에 의해 생성되고 조립된 후, 초기화부터 소멸까지 생명주기를 관리받는 객체를 빈(Bean)이라 부른다.

스프링 컨테이너는 제어의 역전(IoC) 원리가 적용된 스프링의 핵심 컴포넌트다. 빈 구성정보를 바탕으로 POJO 기반 비즈니스 오브젝트를 생성하고, 그 오브젝트들 사이의 의존관계를 주입(DI)하며, 애플리케이션 전체의 생애를 관리한다.

## 빈 구성정보 (Configuration Metadata)

컨테이너가 빈을 생성하고 구성하고 조립할 때 사용하는 설정 정보다. 세 가지 방식으로 작성할 수 있다.

- Java-based configuration: 자바 코드를 빈 설정용 DSL로 사용한다. `@Configuration` 클래스에 `@Bean` 메소드를 선언하는 방식이다. 스프링 3.0부터 지원한다.
- Annotation-based configuration: `@Autowired`, `@Qualifier` 같은 애노테이션으로 작성한다. 스프링 2.5부터 지원한다.
- XML-based configuration: 가장 오래된 방식으로 XML 문서로 작성한다.

```java
@Configuration
public class ContainerConfiguration {

    @Bean
    public MovieFinder movieFinder() {
        return new EmptyMovieFinder();
    }

    @Bean
    public MovieLister movieLister(MovieFinder movieFinder) {
        return new MovieLister(movieFinder);
    }
}
```

`@Configuration`은 이 클래스가 빈 구성정보로 쓰인다는 것을, `@Bean`은 이 메소드가 반환하는 오브젝트를 컨테이너가 등록하고 관리한다는 것을 나타낸다.
`@Bean` 메소드는 파라미터로 다른 빈을 받아 의존관계 주입을 받을 수도 있다.

## 빈 팩토리와 애플리케이션 컨텍스트

빈 팩토리(BeanFactory)는 스프링 IoC를 담당하는 핵심 컨테이너다.
애플리케이션 컨텍스트(ApplicationContext)는 빈 팩토리를 확장해 환경 추상화, 메시지 소스 같은 엔터프라이즈 서비스를 추가로 제공하는 IoC 컨테이너다.
보통 "스프링 컨테이너"라고 하면 애플리케이션 컨텍스트를 가리킨다.

```java
ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(ContainerConfiguration.class);

MovieLister movieLister = applicationContext.getBean(MovieLister.class);
```

## 빈 생명주기

스프링 컨테이너는 빈의 생성부터 초기화, 소멸까지 관여할 수 있는 확장 지점을 제공한다.
자바 표준 공통 애노테이션(JSR-250)인 `@PostConstruct`, `@PreDestroy`를 사용하는 방식이 가장 권장된다.

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateCache() { /* 초기화 로직 */ }

    @PreDestroy
    public void clearCache() { /* 소멸 로직 */ }
}
```

`InitializingBean`, `DisposableBean` 인터페이스를 구현하는 방식도 있지만, 스프링 API에 직접 의존하게 되므로 애노테이션 방식이 더 권장된다.

## 관련 개념

- 참고: ioc.md
- 참고: di.md
- 참고: component-scan.md
