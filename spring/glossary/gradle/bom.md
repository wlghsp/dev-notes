# BOM (Bill of Materials)

여러 라이브러리의 버전을 한곳에서 묶어서 관리하게 해주는 특수한 의존성 선언이다.
Gradle이나 Maven 같은 빌드 도구의 의존성 관리 기능이라 Spring 전용 개념이 아니고, 자바/코틀린 생태계 전반에서 쓰인다.

## 왜 필요한가

어떤 라이브러리를 쓸 때 그 라이브러리 하나만 딱 쓰는 경우는 드물다.
Spring Modulith만 해도 `spring-modulith-starter-jdbc`, `spring-modulith-actuator`, `spring-modulith-observability`, `spring-modulith-starter-test`처럼 서로 연관된 여러 모듈을 함께 쓴다.
이 모듈들을 각각 버전을 따로 명시하면, 버전이 하나라도 안 맞을 때 호환성 문제가 생길 수 있다.
예를 들어 `spring-modulith-actuator`는 1.3.4인데 `spring-modulith-starter-jdbc`는 실수로 1.2.0을 쓰면, 두 모듈이 서로 다른 버전에서 설계된 API를 기대하면서 런타임에 예상치 못한 오류가 날 수 있다.

BOM은 이 문제를 "버전을 여러 곳에 흩어놓지 말고 한곳에서만 정하자"는 방식으로 해결한다.
BOM 자체는 실제 코드가 든 라이브러리가 아니라, "이 그룹에 속한 모듈들은 이 버전 조합으로 쓰는 게 맞다"는 버전 정보만 담은 메타데이터다.

## 사용 방식

```kotlin
// build.gradle.kts
dependencyManagement {
    imports {
        mavenBom("org.springframework.modulith:spring-modulith-bom:1.3.4")
    }
}

dependencies {
    implementation("org.springframework.modulith:spring-modulith-starter-jdbc")
    runtimeOnly("org.springframework.modulith:spring-modulith-actuator")
}
```

`mavenBom`으로 BOM을 한 번 가져오면, 그 아래 `dependencies` 블록에서는 `spring-modulith-starter-jdbc`처럼 모듈 이름만 적고 버전을 따로 쓰지 않아도 된다.
BOM이 정해둔 버전이 자동으로 적용되기 때문이다.
버전을 올릴 때도 BOM 좌표의 버전 문자열(`1.3.4`) 하나만 바꾸면, 그 BOM에 속한 모든 모듈의 버전이 한 번에 맞춰 올라간다.

## Spring Boot에서 이미 익숙하게 쓰고 있던 개념

사실 Spring Boot 프로젝트를 만들 때 `spring-boot-starter-web`, `spring-boot-starter-data-jpa` 같은 스타터에 버전을 안 적어도 되는 이유가 바로 이 BOM 메커니즘이다.
Spring Boot Gradle 플러그인이 `spring-boot-dependencies`라는 BOM을 내부적으로 이미 적용해주기 때문에, 개발자가 직접 `mavenBom`을 호출하지 않아도 각 스타터의 버전이 자동으로 맞춰진다.
Spring Modulith는 Spring Boot 플러그인이 자동으로 관리해주는 대상이 아니라서, `spring-modulith-bom`을 직접 명시적으로 가져와야 한다.

## 관련 개념

BOM으로 가져온 라이브러리를 실제로 `dependencies` 블록에 넣을 때, `implementation`인지 `runtimeOnly`인지 `testImplementation`인지에 따라 그 라이브러리가 언제 코드에 보이고 언제 실행되는지가 달라진다. 이 구분은 dependency-configuration.md에서 다룬다.
