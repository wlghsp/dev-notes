# dependencyManagement 블록과 Dependency Management 플러그인

Gradle에서 `dependencyManagement { }` 블록은 io.spring.dependency-management라는 Gradle 플러그인이 제공하는 문법이다.
이 블록 안에서 `imports { mavenBom(...) }`으로 BOM을 가져오거나, 특정 라이브러리의 버전을 직접 지정해서 프로젝트 전체에 적용할 수 있다.

## Gradle 기본 기능이 아니다

`dependencyManagement`는 Gradle 자체가 기본으로 제공하는 문법이 아니라, Spring Boot Gradle 플러그인이 내부적으로 함께 적용해주는 io.spring.dependency-management 플러그인의 문법이다.
Spring Boot 프로젝트가 아니거나 이 플러그인을 직접 추가하지 않은 순수 Gradle 프로젝트에서는 이 블록 문법을 쓸 수 없다.
Gradle에 원래 내장된, BOM을 가져오는 표준 방식은 `implementation(platform("..."))`처럼 `platform()` 함수를 쓰는 것이다.

```kotlin
// io.spring.dependency-management 플러그인 방식
dependencyManagement {
    imports {
        mavenBom("org.springframework.modulith:spring-modulith-bom:1.3.4")
    }
}

// Gradle 표준 platform() 방식 (같은 목적, 다른 문법)
dependencies {
    implementation(platform("org.springframework.modulith:spring-modulith-bom:1.3.4"))
}
```

## 왜 Spring 프로젝트에서는 dependencyManagement 방식을 더 자주 보게 되는가

Spring Boot Gradle 플러그인을 프로젝트에 적용하면 io.spring.dependency-management 플러그인이 함께 딸려 들어오는 경우가 많다.
그래서 Spring 생태계 예제나 블로그 글에서는 `platform()`보다 `dependencyManagement` 블록 문법이 더 자주 보인다.
기능적으로는 두 방식 모두 "이 BOM이 정한 버전을 프로젝트 의존성에 적용한다"는 같은 목적을 수행하고, 어느 쪽을 쓸지는 프로젝트에 어떤 플러그인이 이미 적용돼 있는지에 달려 있다.

## 관련 개념

이 블록이 가져오는 대상인 BOM 자체의 개념은 bom.md에서 다룬다.
