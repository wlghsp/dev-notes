# 클래스 로더 (ClassLoader)

JVM이 `.class` 파일을 찾아 메모리에 적재하고 실행 가능한 상태로 만드는 역할을 한다. 이 과정은 크게 Loading, Linking, Initialization 세 단계로 나뉜다.

## Loading

클래스 파일을 가져와 JVM 메모리에 적재하는 단계다. 이 단계는 로더의 역할에 따라 다시 셋으로 나뉜다.

Bootstrap Class Loading은 JVM 기본 클래스와 자바 코드를 로딩한다. Extension Class Loading은 자바 핵심 라이브러리를 로딩한다. Application Class Loading은 개발자가 작성해 classpath에 올려둔 클래스를 로딩한다.

## Linking

클래스가 참조하는 다른 클래스, 메서드, 필드를 확인하고 필요하면 메모리 상에서 연결하는 단계다. Verification, Prepare, Resolution 세 단계로 나뉜다.

## Initialization

클래스 변수를 초기화하거나 static 블록 내의 코드를 실행하는 단계다.

## Lazy Loading

클래스 로더는 애플리케이션이 시작될 때 모든 클래스를 미리 로딩하지 않는다. 대신 클래스가 최초로 필요해지는 시점까지 로딩을 지연한다.

이 방식 때문에 배포 직후에는 대부분의 클래스가 아직 메모리에 적재되지 않은 상태다. 그 상태에서 요청이 들어오면 클래스 로더가 그제서야 클래스를 적재하므로, 이 적재 비용이 그대로 응답 지연으로 나타난다. 참고: jvm-warm-up.md
