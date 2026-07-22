# Dependency Configuration (implementation / runtimeOnly / testImplementation)

Gradle에서 어떤 의존성을 `dependencies` 블록에 선언할 때, 그 의존성이 언제(컴파일 시점, 실행 시점, 테스트 시점) 어디에 노출되는지를 구분하는 설정이다.
같은 라이브러리라도 이 configuration을 무엇으로 쓰느냐에 따라 프로젝트에 미치는 영향이 달라진다.

## 왜 구분이 필요한가

의존성을 무조건 한 가지 방식(예를 들어 전부 `implementation`)으로만 선언하면 두 가지 문제가 생긴다.
하나는 컴파일 속도다. 실제로는 실행할 때만 필요하고 코드에서 직접 참조하지 않는 라이브러리까지 컴파일 타임에 끌고 오면, 코드가 바뀔 때마다 재컴파일해야 할 대상이 불필요하게 늘어난다.
다른 하나는 의존성 누출이다. 이 모듈을 다른 모듈이 가져다 쓸 때, 내가 내부적으로만 쓰는 라이브러리까지 그 모듈에 전이되어 노출되면 원치 않는 결합이 생긴다.

## 주요 configuration

`implementation`은 가장 일반적으로 쓰는 설정이다.
컴파일 시점과 실행 시점 모두에 포함되고, 코드에서 이 라이브러리의 클래스를 직접 import해서 쓸 수 있다.
다만 이 모듈을 다른 모듈이 의존성으로 가져다 쓰더라도, `implementation`으로 선언한 라이브러리는 그 다른 모듈에는 전이되지 않는다.
여기어때 글의 예시에서 `spring-modulith-starter-jdbc`가 `implementation`인 이유는, `ApplicationModuleListener` 같은 타입을 코드에서 직접 참조해서 쓰기 때문이다.

`runtimeOnly`는 컴파일 시점에는 필요 없고 실행 시점에만 필요한 라이브러리에 쓴다.
코드에서 이 라이브러리의 클래스를 직접 import하지 않지만, 애플리케이션이 동작하려면 클래스패스에 존재해야 하는 경우다.
여기어때 글에서 `spring-modulith-actuator`와 `spring-modulith-observability`가 `runtimeOnly`인 이유는, 이 두 라이브러리가 `/actuator/modulith` 엔드포인트나 모니터링 연동을 자동으로 붙여주는 역할이라 코드에서 직접 그 클래스를 호출할 일이 없기 때문이다.
컴파일 시점에 필요 없는 라이브러리를 `runtimeOnly`로 명시하면, 그 라이브러리가 없어도 코드가 컴파일되는지(즉 코드가 이 라이브러리에 실수로 컴파일 의존을 갖고 있지 않은지)까지 빌드 도구가 검증해준다.

`testImplementation`은 테스트 코드에서만 필요한 라이브러리에 쓴다.
`main` 소스셋에는 포함되지 않고 `test` 소스셋에만 포함되므로, 실제 배포되는 애플리케이션에는 이 라이브러리가 들어가지 않는다.
여기어때 글에서 `spring-modulith-starter-test`가 `testImplementation`인 이유는, `verify()`나 `Documenter`, `Scenario` API가 아키텍처 검증과 테스트 용도이지 운영 중인 애플리케이션이 실행 시점에 쓸 코드가 아니기 때문이다.

## 한눈에 구분하는 기준

이 라이브러리의 클래스를 내 코드(main)에서 직접 import해서 쓰는가, 그리고 이게 테스트에서만 필요한가 두 가지를 물어보면 된다.
main 코드에서 직접 쓰면 `implementation`, main 코드에서 직접 쓰지 않지만 실행에는 필요하면 `runtimeOnly`, 테스트 코드에서만 쓰면 `testImplementation`이다.

## 관련 개념

이 configuration들에 버전을 명시하지 않고도 쓸 수 있게 해주는 메커니즘이 BOM이다. bom.md 참고.
