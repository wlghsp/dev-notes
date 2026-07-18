# 이식 가능한 서비스 추상화 (Portable Service Abstraction)

환경과 구현 기술의 변경과 무관하게 일관된 방식으로 엔터프라이즈 기술을 다룰 수 있게 지원하는 스프링의 원리다.
제어의 역전(IoC) 원리를 통해 POJO에게 제공된다. 트랜잭션(transaction), 캐시(cache), 메일(mail), 메시징(messaging), Object/XML Mapping(OXM) 등 다양한 기술의 서비스 추상화가 제공된다.

핵심은 비즈니스 오브젝트(POJO)가 구체적인 구현 기술을 몰라도 되게 만드는 것이다.
예를 들어 XmlMovieReader라는 비즈니스 오브젝트는 자바 객체와 XML을 변환할 때 Object-XML 매퍼라는 추상화를 사용한다.
실제 변환 작업을 JAXB, JiBX, XStream 중 무엇이 하는지는 XmlMovieReader가 알 필요가 없다.
구현 기술이 바뀌어도 비즈니스 오브젝트는 영향을 받지 않는다.

## Resource와 ResourceLoader 추상화

`Resource` 인터페이스는 저수준 리소스 접근을 추상화한다.
자바 표준 `java.net.URL` 클래스와 그 접두사들은 다양한 저수준 리소스에 접근하기에 부족해서, 스프링이 이 문제를 해결하려고 만들었다.
클래스패스(`ClassPathResource`), 파일(`FileSystemResource`), HTTP(`UrlResource`), 서블릿 컨텍스트(`ServletContextResource`) 등 내장된 구현체를 제공한다.

```java
Resource resource = applicationContext.getResource("classpath:template.tml");
Resource resource = applicationContext.getResource("file:./files/template.tml");
Resource resource = applicationContext.getResource("https://myhost.com/resource/template.tml");
```

`ResourceLoader` 인터페이스 구현체는 주어진 위치에서 `Resource` 객체를 반환(로딩)한다.
애플리케이션 컨텍스트는 이 `ResourceLoader` 인터페이스를 확장하고 있어서, 접두사만 바꾸면 클래스패스든 파일이든 URL이든 동일한 방식으로 자원을 불러올 수 있다.
접두사를 지정하지 않으면 애플리케이션 컨텍스트 타입에 따라 자원을 불러온다. 스프링 웹 환경에서는 기본적으로 서블릿 자원을 찾는다.

## 데이터 접근 서비스 추상화

데이터베이스나 구현 기술에 종속적인 반복 코드를 없애주는 추상화다.

JDBC API를 직접 사용하면 커넥션 획득, SQL 실행, 예외 처리, 커넥션 종료, 트랜잭션 제어를 모두 손으로 작성해야 한다.
스프링이 제공하는 `JdbcTemplate`과 트랜잭션 매니저를 쓰면 이 반복적인 코드를 걷어낼 수 있다.

```java
TransactionStatus transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());
try {
    jdbcTemplate.update(sql, member.getId(), member.getName());
    transactionManager.commit(transaction);
} catch (Exception error) {
    transactionManager.rollback(transaction);
    throw error;
}
```

일관된 프로그래밍 모델로 트랜잭션을 제어할 수 있고, 데이터베이스나 구현 기술별로 달라지는 예외도 스프링이 정의한 일관된 예외 계층(`DataAccessException` 등)으로 다룰 수 있다.

## 관련 개념

- 참고: ioc.md
- 참고: bean.md
