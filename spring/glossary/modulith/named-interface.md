# Named Interface

Spring Modulith에서 CLOSED 타입 모듈이 특정 내부 패키지만 선택적으로 외부에 공개할 때 쓰는 선언이다.
`@org.springframework.modulith.NamedInterface` 어노테이션을 패키지 단위(`package-info.java` 또는 Kotlin의 `package-info` 형태)에 붙여서 만든다.

## 왜 필요한가

`@ApplicationModule`의 기본값인 Type.CLOSED는 모듈 루트 패키지의 public 타입만 외부에 노출한다.
그런데 모듈 내부에도 "이건 숨기고 싶은 세부 구현"과 "이건 다른 모듈에 열어주고 싶은 포트"가 섞여 있는 경우가 많다.
예를 들어 `roomrate` 모듈의 `application/port/` 패키지는 다른 모듈이 정상적으로 호출해야 하는 인터페이스지만, 모듈 루트 패키지가 아니라 하위 패키지에 있어서 CLOSED 기본 규칙으로는 접근이 막힌다.
이럴 때 그 패키지에 Named Interface를 선언하면 예외적으로 외부 접근을 허용할 수 있다.

## 사용 방식

```
// roomrate 모듈의 application/port/ 패키지에 package-info 형태로 선언
@org.springframework.modulith.NamedInterface("ports")
package kr.co.gccompany.stay.product.ari.roomrate.application.port
```

이렇게 선언하면 다른 모듈에서는 `roomrate::ports`라는 이름으로만 이 패키지에 접근하도록 강제할 수 있다.
`allowedDependencies`에서도 모듈 전체가 아니라 이 named interface 단위로 의존을 지정할 수 있다.

## MSA와의 대응 관계

MSA에서 서비스가 API만 외부에 노출하고 내부 구현은 숨기는 것과 동일한 효과를, 하나의 애플리케이션 내부에서 패키지 단위로 얻을 수 있는 것이 Named Interface의 핵심이다.
모듈 루트 패키지의 public 타입 전부를 공개 API로 쓰지 않고, 정말 공개하고 싶은 하위 패키지만 골라서 열 수 있다는 점에서 CLOSED 타입의 기본 규칙보다 더 세밀한 통제가 가능하다.
