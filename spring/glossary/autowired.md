# @Autowired와 빈 프로퍼티 값 설정

## @Autowired를 사용한 의존관계 설정

빈 타입에 의한 자동와이어링 방식으로 빈 의존관계를 설정하는 애노테이션이다.
생성자, 수정자 메소드, 일반 메소드, 필드에 선언해서 사용한다.

```java
@Component
class MovieLister {

    private MovieFinder finder;

    @Autowired
    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
}
```

생성자에 `@Autowired`를 선언하면 빈 생성 시 의존관계를 설정한다. 스프링 4.3 이상에서는 생성자가 하나뿐이면 이 애노테이션도 생략할 수 있다.
내부적으로는 스프링 컨테이너의 확장 수단인 `BeanPostProcessor`(정확히는 `AutowiredAnnotationBeanPostProcessor`)를 통해 동작한다.
자바 표준인 JSR-250의 `@Resource`, JSR-330의 `@Inject`도 같은 목적으로 사용할 수 있다.

## @Value를 사용한 프로퍼티 값 설정

`@Value`는 환경 설정정보 또는 다른 빈을 이용해 빈 프로퍼티 값을 설정하는 애노테이션이다. 필드, 설정자 메소드, 메소드 파라미터에 사용한다.

```java
@Value("${catalog.name}")
public void setCatalog(String catalog) {
    this.recommendedCategory = catalog;
}
```

`${ }` 안에 키를 적으면 프로퍼티 소스에서 그 키에 연결된 값을 읽어 설정한다. 이것을 프로퍼티 치환자라 부른다.
`${catalog.name:defaultCatalog}`처럼 콜론 뒤에 기본값을 적으면, 연결된 값이 없을 때 그 기본값이 쓰인다.

프로퍼티 치환자가 동작하려면 `PropertySourcesPlaceholderConfigurer` 빈을 정적(static) 메소드로 등록해야 한다.

```java
@Bean
public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
    return new PropertySourcesPlaceholderConfigurer();
}
```

## 스프링 표현 언어(SpEL)로 값 설정하기

스프링 표현 언어(줄여서 SpEL)는 런타임 시에 객체 그래프를 조회하고 조작하는 스프링 전용 표현식 언어다.
`#{ }` 안에 작성된 표현식의 평가 결과를 빈 프로퍼티 값으로 설정한다.

```java
@Value("#{applicationProperties['catalog.name'] ?: 'defaultCatalog'}")
public void setCatalog(String catalog) {
    this.catalog = catalog;
}
```

`?:`는 엘비스 오퍼레이터(Elvis Operator)로, 키에 연결된 값이 없으면 오른쪽 기본값을 사용한다.
프로퍼티 치환자(`${ }`)와 달리 SpEL(`#{ }`)은 다른 빈 객체의 프로퍼티에 접근하거나 다양한 연산을 수행할 수 있다.

## 관련 개념

- 참고: bean.md
- 참고: component-scan.md
- 참고: environment.md
