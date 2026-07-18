# 환경 추상화 (Environment Abstraction)

실행 환경에 따라 빈 구성을 다르게 할 수 있고, 일관된 방식으로 외부 설정정보를 관리하고 접근할 수 있게 해주는 스프링의 서비스 추상화다.
프로파일(profile)과 프로퍼티 소스(property-source)라는 두 축으로 구성되며, 컨테이너와 통합되어 있다.

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();

applicationContext.getEnvironment().setActiveProfiles("development", "cloud");

Environment environment = applicationContext.getEnvironment();
boolean containsJavaVersion = environment.containsProperty("java.version");
String javaVersion = environment.getProperty("java.version", String.class);
```

## 빈 정의 프로파일 (Profiles)

실행 환경에 따라 컨테이너에 등록할 빈을 다르게 선택하는 매커니즘이다. `@Profile` 애노테이션으로 프로파일별 빈 등록을 지정한다.

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.HSQL)
                .addScript("classpath:/db/schema.sql")
                .build();
    }
}
```

프로파일은 세 가지 방법으로 활성화할 수 있다.

- OS 환경변수: `export spring_profiles_active=development`
- JVM 시스템 파라미터: `java -jar -Dspring.profiles.active=development application.jar`
- 서블릿 배포서술자(web.xml)의 `<context-param>`

## 프로퍼티 소스 추상화 (PropertySource Abstraction)

키=밸류 형태로 작성된 설정정보를 프로퍼티 소스라 부른다. 보통 애플리케이션 외부에서 불러와 구성한다.
OS 환경변수, JVM 시스템 프로퍼티, 커맨드라인 인수, 서블릿 설정/매개변수, JNDI 환경변수, 프로퍼티 파일(.properties, .yml) 등 다양한 소스를 일관된 방식으로 사용할 수 있게 지원한다.

```java
@Configuration
@PropertySource("classpath:/application.properties")
public class UsingPropertySourceExample {

    @Autowired
    private Environment env;
}
```

`@PropertySource` 애노테이션은 프로퍼티 파일(.properties)을 읽어 스프링 환경(Environment)에 등록한다.
이렇게 등록된 값은 `@Value`의 프로퍼티 치환자(`${ }`)로 빈 프로퍼티 값에 주입할 수 있다.

## 관련 개념

- 참고: bean.md
- 참고: autowired.md
