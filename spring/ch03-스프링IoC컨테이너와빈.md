# 제2편 3장, 스프링 IoC 컨테이너와 빈

이 챕터에서 생성된 키워드 파일: bean.md, component-scan.md, autowired.md, environment.md, message-source.md

원본: 스프링 프로그래밍 배우기 (springrunner.dev)

---

## 이 장에서 다루는 것

2장에서 다룬 제어의 역전(IoC)과 의존관계 주입(DI)이라는 원리를, 스프링이 실제로 어떤 컨테이너와 API로 구현하는지 다루는 장이다.
빈을 등록하는 여러 방법부터 빈의 생명주기, 자동 의존관계 설정, 환경별 구성, 국제화까지 스프링 컨테이너가 제공하는 핵심 기능들을 폭넓게 훑는다.

## 스프링 IoC 컨테이너와 빈

컨테이너에 의해 생성되고 조립된 후 관리되는 객체를 빈(Bean)이라 부른다.
컨테이너는 빈 구성정보(Configuration Metadata)를 바탕으로 POJO 기반 비즈니스 오브젝트를 만들고 의존관계를 주입하며 생애를 관리한다.

빈 구성정보를 작성하는 방식은 세 가지다. 자바 코드로 작성하는 Java-based configuration, 애노테이션으로 작성하는 Annotation-based configuration, 가장 오래된 XML-based configuration이다.
이 문서(원본 슬라이드)는 자바 코드 방식을 중심으로 다룬다.

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

빈 팩토리(BeanFactory)는 스프링 IoC를 담당하는 핵심 컨테이너이고, 애플리케이션 컨텍스트(ApplicationContext)는 빈 팩토리를 확장해 엔터프라이즈 서비스를 추가로 제공하는 컨테이너다.
보통 스프링 컨테이너라고 하면 애플리케이션 컨텍스트를 가리킨다. 자세한 내용은 bean.md에 정리했다.

## 자동 클래스 탐지로 빈 등록하기

`@Bean` 메소드를 하나씩 작성하는 대신, 클래스에 `@Component` 같은 스테레오타입 애노테이션을 붙이고 `@ComponentScan`으로 컨테이너에게 탐지를 맡길 수 있다.

```java
@Configuration
@ComponentScan("tutorial.example")
public class ComponentScanConfiguration {
}
```

`@Service`, `@Repository`, `@Controller`는 모두 `@Component`의 특수한 형태로, 컨테이너 입장에서는 동일하게 동작하지만 어떤 계층의 클래스인지 이름으로 구분할 수 있게 해준다.
자세한 내용은 component-scan.md에 정리했다.

## 빈 생명주기 관여하기

스프링 컨테이너는 빈의 생성부터 초기화, 소멸까지 관여할 수 있는 확장 지점을 제공한다.
JSR-250 표준 애노테이션인 `@PostConstruct`, `@PreDestroy`를 사용하는 방식이 가장 권장된다.

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateCache() { /* ... */ }

    @PreDestroy
    public void clearCache() { /* ... */ }
}
```

## @Autowired와 프로퍼티 값 설정

`@Autowired`는 빈 타입에 의한 자동와이어링으로 의존관계를 설정하는 애노테이션이다. 생성자, 수정자 메소드, 필드에 선언한다.
`@Value`는 환경 설정정보나 다른 빈을 이용해 빈 프로퍼티 값을 설정한다. `${ }` 프로퍼티 치환자와 `#{ }` 스프링 표현 언어(SpEL) 두 가지 문법을 지원한다.

```java
@Value("${catalog.name:defaultCatalog}")
public void setCatalog(String catalog) {
    this.catalog = catalog;
}
```

`:` 뒤의 값은 프로퍼티 소스에 해당 키가 없을 때 쓰이는 기본값이다. 자세한 내용은 autowired.md에 정리했다.

## 환경 추상화

실행 환경에 따라 빈 구성을 다르게 하고, 일관된 방식으로 외부 설정정보를 다루게 해주는 추상화다. 프로파일(profile)과 프로퍼티 소스(property-source)로 구성된다.

`@Profile` 애노테이션으로 개발/운영 같은 환경별로 다른 빈을 등록할 수 있다.
OS 환경변수, JVM 시스템 프로퍼티, 서블릿 매개변수, 프로퍼티 파일 등 다양한 소스를 `Environment` API 하나로 일관되게 조회할 수 있다.
자세한 내용은 environment.md에 정리했다.

## MessageSource로 국제화 메시지 다루기

`MessageSource` 인터페이스는 국제화(i18n) 메시지를 다룬다. 애플리케이션 컨텍스트는 이 인터페이스를 확장하고 있다.
로케일(Locale)에 따라 다른 언어의 메시지를 돌려줄 수 있어서, 에러 메시지나 화면 문구를 다국어로 관리해야 할 때 쓴다. 자세한 내용은 message-source.md에 정리했다.

## 정리

이 장은 2장에서 배운 IoC와 DI라는 원리가 스프링 컨테이너 안에서 구체적으로 어떻게 구현되는지를 보여준다.
빈을 어떻게 등록하고(`@Bean`, `@ComponentScan`), 어떻게 의존관계를 주입받고(`@Autowired`, `@Value`), 어떻게 환경에 따라 다르게 구성하고(`Environment`, `@Profile`), 어떻게 다국어를 지원하는지(`MessageSource`)까지가 컨테이너가 제공하는 핵심 기능들이다.
다음 장(이식 가능한 서비스 추상화)은 트랜잭션, 캐시, 메일 같은 더 구체적인 엔터프라이즈 서비스를 스프링이 어떻게 추상화하는지를 다룬다.
