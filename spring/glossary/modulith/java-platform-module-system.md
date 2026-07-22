# Java Platform Module System (JPMS)

Java 9부터 도입된 언어 자체의 모듈 시스템이다.
`module-info.java`라는 파일로 모듈이 무엇을 외부에 노출(export)하고 무엇을 필요로(requires) 하는지 선언하면, 자바 컴파일러와 런타임이 이 규칙을 강제한다.

## Spring Modulith의 verify()와 다른 지점

Spring Modulith의 `ApplicationModules.verify()`는 테스트를 실행해야 위반 여부를 알 수 있는, 테스트 타임 검증이다.
반면 JPMS는 언어 자체의 기능이라 컴파일 타임에 강제된다.
`module-info.java`에 `exports`로 선언하지 않은 패키지의 클래스를 다른 모듈에서 import하려고 하면, 테스트를 실행할 필요도 없이 컴파일 자체가 실패한다.
즉 JPMS는 더 이른 시점에, 더 근본적인 수준(언어 차원)에서 모듈 경계를 강제한다는 차이가 있다.

## 사용 방식

```java
// catalog 모듈의 module-info.java
module coffeehouse.modules.catalog {
    requires coffeehouse.libraries.base;
    exports coffeehouse.modules.catalog.domain;
    exports coffeehouse.modules.catalog.domain.service;
}
```

`requires`는 이 모듈이 의존하는 다른 모듈을 선언하고, `exports`는 이 모듈에서 외부에 공개할 패키지를 선언한다.
`exports`로 선언되지 않은 패키지(예를 들어 데이터 접근 구현체가 있는 패키지)는 다른 모듈이 아무리 클래스패스에 그 클래스가 존재해도 참조할 수 없다.

## 왜 잘 안 쓰이는가

JPMS는 이론적으로 가장 강력한 캡슐화 수단이지만, 실무에서는 Spring Modulith의 `verify()`나 단순 패키지 가시성(default/package-private)만큼 자주 쓰이지 않는다.
Spring을 포함한 많은 자바 생태계 라이브러리가 리플렉션(reflection)에 의존해서 동작하는데, JPMS의 엄격한 캡슐화가 이런 리플렉션 기반 프레임워크와 충돌하는 경우가 있다.
그래서 JPMS를 도입하려면 모듈 선언을 프레임워크 요구사항에 맞게 세심하게 조정해야 하는 진입 장벽이 있고, 이 비용 때문에 Spring Modulith의 테스트 기반 검증이나 패키지 가시성 제어로 대신하는 경우가 많다.

## 가시성 수준과의 관계

JPMS를 쓰지 않더라도 자바의 기본 가시성 제어(`public`, `protected`, `default(package)`, `private`)만으로도 모듈 내부 구현을 어느 정도 보호할 수 있다.
`public`으로 선언한 것만 모듈 경계 밖에서 접근 가능하게 하고, 나머지는 `default`나 `package-private`으로 감추는 방식이다.
JPMS는 이 가시성 규칙을 패키지 단위로 한 번 더 강제하는 상위 계층이라고 이해하면 된다.
