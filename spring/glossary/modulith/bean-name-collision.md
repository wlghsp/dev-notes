# 모듈러 모놀리스의 Bean 이름 충돌

Spring Modulith로 모듈을 나눠도 여러 모듈이 하나의 Spring ApplicationContext를 공유하기 때문에 발생하는 문제다.
서로 다른 모듈이 같은 클래스명을 쓰면 Bean 등록 단계에서 이름이 겹쳐 애플리케이션 부팅이 실패한다.

## 왜 발생하는가

Spring Modulith가 강제하는 것은 모듈 간 참조 규칙(어떤 패키지를 어떤 모듈이 접근할 수 있는가)이지, Bean 이름 공간을 모듈별로 분리해주는 게 아니다.
MSA라면 서비스마다 별도의 프로세스와 별도의 Spring Context를 가지므로 같은 클래스명이 있어도 충돌하지 않는다.
하지만 모듈러 모놀리스는 배포 단위가 하나이고 Context도 하나이기 때문에, 예를 들어 `ari` 모듈과 `overseas` 모듈이 각각 `RoomRateCommandService`라는 이름의 클래스를 갖고 있으면 Bean 등록 시점에 이름이 겹친다.

## 해결 방식

커스텀 `BeanNameGenerator`를 만들어서, Bean을 등록할 때 클래스가 속한 모듈에 따라 접두사를 붙이는 방식으로 해결한다.

```kotlin
@Modulithic
@ComponentScan(nameGenerator = AriBeanNameGenerator::class)
@Configuration
class AriAutoConfig

class AriBeanNameGenerator : BeanNameGenerator {
    override fun generateBeanName(
        definition: BeanDefinition,
        registry: BeanDefinitionRegistry
    ): String {
        val defaultName = defaultGenerator.generateBeanName(definition, registry)
        return if (definition.beanClassName!!.contains(".ari.")) {
            "ari${defaultName.replaceFirstChar { it.uppercaseChar() }}"
        } else defaultName
    }
}
```

이렇게 하면 `ari` 모듈의 Bean은 `ariRoomRateCommandService`, `overseas` 모듈의 Bean은 `overseasRoomRateCommandService`로 등록되어 이름이 겹치지 않는다.

## 실무적 의미

이 문제는 모듈 수가 늘어날수록 발생 확률이 높아지는 구조적인 문제라, 모듈이 몇 개 없을 때는 안 보이다가 나중에 갑자기 터지는 경우가 많다.
그래서 Bean 이름 생성 규칙은 모듈러 모놀리스 도입 초기, 모듈 수가 적을 때 미리 정해두는 것이 실무적으로 안전하다.
나중에 모듈이 쌓인 뒤에 규칙을 바꾸면 기존 Bean 이름을 참조하는 코드(설정, 테스트 등)를 일괄로 고쳐야 하는 비용이 크다.
