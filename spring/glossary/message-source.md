# MessageSource와 국제화 메시지

`MessageSource` 인터페이스는 국제화(internationalization, 줄여서 i18n) 스타일의 메시지를 다루는 스프링 추상화다.
표준 JDK의 `ResourceBundle`과 같은 로케일(locale) 처리 방법과 장애복구 규칙을 따른다.
애플리케이션 컨텍스트(ApplicationContext)는 이 `MessageSource` 인터페이스를 확장하고 있어서, 컨테이너를 통해 바로 메시지를 조회할 수 있다.

```java
@Configuration
class MessageSourceExample {

    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasenames("format", "exceptions");
        return messageSource;
    }
}
```

```
# format.properties
message=Alligators rock!

# exceptions.properties
argument.required=The '{0}' argument is required.

# exceptions_ko_KR.properties
argument.required='{0}' 은 필수 값 입니다.
```

`getMessage(String code, Object[] args, String default, Locale locale)`가 메시지를 획득할 때 쓰는 기본 메소드다.
코드에 해당하는 메시지를 찾지 못하면 기본 메시지를 대신 사용한다.
전달한 아규먼트는 표준 JDK의 `MessageFormat` 기능으로 메시지 안의 `{0}` 같은 자리에 치환된다.
전달된 로케일(예: `Locale.KOREA`)에 따라 `exceptions_ko_KR.properties`처럼 로케일이 붙은 프로퍼티 파일에서 메시지를 찾는다.

이 기능이 필요한 이유는 하드코딩된 문자열 대신 코드와 로케일만으로 사용자 언어에 맞는 메시지를 돌려줄 수 있기 때문이다.
에러 메시지, 화면에 노출되는 문구를 다국어로 관리해야 하는 애플리케이션에서 유용하다.

## 관련 개념

- 참고: environment.md
