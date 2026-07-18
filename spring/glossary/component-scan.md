# 컴포넌트 스캔 (Component Scan)

지정된 클래스패스에서 스테레오타입(stereotyped) 애노테이션이 붙은 클래스를 자동으로 탐지해 빈으로 등록하는 기능이다.

`@Bean` 메소드를 하나씩 작성하는 대신, 클래스 자체에 애노테이션을 붙여두면 컨테이너가 알아서 찾아 등록한다.

```java
@Configuration
@ComponentScan("tutorial.example")
public class ComponentScanConfiguration {
}
```

```java
package tutorial.example;

@Component
public class EmptyMovieFinder implements MovieFinder {
    // 생략
}
```

`@ComponentScan`을 선언한 패키지(`tutorial.example`) 이하에서 스테레오타입 애노테이션이 붙은 클래스를 탐지하고 빈으로 등록한다.

## 스테레오타입 애노테이션

- `@Component`: 스프링이 관리하는 모든 컴포넌트에 대한 제너릭 스테레오타입이다.
- `@Service`: `@Component`의 특수한 형태로 서비스 계층에서 사용하면 적합하다.
- `@Repository`: `@Component`의 특수한 형태로 데이터 접근 계층에서 사용하면 적합하다.
- `@Controller`: `@Component`의 특수한 형태로 프리젠테이션 계층에서 사용하면 적합하다.

넷 다 컨테이너 입장에서는 동일하게 빈으로 등록되지만, 이름을 구분해두면 코드를 읽을 때 그 클래스가 어떤 계층에 속하는지 바로 알 수 있다.

## 관련 개념

- 참고: bean.md
- 참고: autowired.md
