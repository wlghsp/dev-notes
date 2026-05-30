# classloading
참고: Java Performance: The Definitive Guide Ch.12 (Classloading)

---

JVM이 .class 파일을 찾아서 메모리에 올리는 과정이다. `new`로 객체를 만들거나, static 멤버를 처음 참조할 때 해당 클래스가 아직 로드되지 않았다면 클래스 로더가 동작한다.

---

## 클래스 로딩의 세 단계

1. Loading — .class 파일을 찾아서 바이트코드를 메모리에 읽어들인다
2. Linking — 바이트코드 검증(verification), 기본값 초기화(preparation), 심볼릭 참조 해결(resolution)
3. Initialization — static 초기화 블록과 static 필드 초기화 코드를 실행한다

---

## 클래스 로더 계층 구조

클래스 로더는 세 계층으로 구성된다. 부모에게 먼저 위임하고, 부모가 못 찾으면 자신이 찾는 위임 모델(parent delegation model)을 따른다.

1. Bootstrap ClassLoader — JVM 자체에 내장. `java.lang`, `java.util` 등 JDK 핵심 클래스 로드
2. Extension ClassLoader — `$JAVA_HOME/lib/ext` 경로의 클래스 로드
3. Application ClassLoader — 애플리케이션 classpath의 클래스 로드. 우리가 작성한 코드가 여기서 로드됨

---

## 클래스 로딩과 성능

클래스는 처음 참조될 때 로드된다(lazy loading). 애플리케이션 시작 시점에 많은 클래스가 한꺼번에 로드되면 초기 구동 시간이 느려진다.

클래스 로딩은 Metaspace(Java 8 이후) 또는 PermGen(Java 7 이하)을 소비한다. 동적으로 클래스를 생성하는 프레임워크(리플렉션, AOP, 코드 생성 라이브러리)는 클래스를 대량으로 생성하기 때문에 Metaspace가 부족해질 수 있다.

로드된 클래스는 해당 클래스 로더가 GC 수집 대상이 될 때만 언로드된다. 일반적으로 클래스는 JVM 종료 전까지 메모리에 유지된다.
