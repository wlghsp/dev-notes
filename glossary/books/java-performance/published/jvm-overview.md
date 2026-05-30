# jvm-overview
참고: Java Performance: The Definitive Guide Ch.1, 4

---

JVM(Java Virtual Machine)은 Java 바이트코드를 실행하는 프로세스다.

"가상 머신"이라는 이름처럼, 실제 하드웨어 위에서 동작하지만 프로그램 입장에서는 일관된 가상 환경을 제공한다. 덕분에 Java 코드는 어떤 OS, 어떤 CPU 아키텍처에서도 동일하게 동작한다. Write Once, Run Anywhere.

---

## JVM이 하는 일

JVM은 단순한 실행 엔진이 아니다. 다음 세 가지를 동시에 담당한다.

1. 실행 — 바이트코드를 해석(interpret)하거나 기계어로 컴파일(JIT)해서 실행한다
2. 메모리 관리 — 객체를 힙에 할당하고, 더 이상 참조되지 않는 객체를 GC로 회수한다
3. 안전 보장 — 배열 경계 검사, null 체크, 타입 안전성을 런타임에 보장한다

---

## JVM은 하나의 프로세스다

`java` 명령어를 실행하면 OS 입장에서는 하나의 프로세스가 시작된다. 이 프로세스 안에 여러 스레드가 동작하고, 힙·스택·코드 캐시 등 여러 메모리 영역이 존재한다.

JVM이 "느리다"는 편견은 초창기 인터프리터 방식에서 비롯됐다. 현대 JVM은 JIT 컴파일과 적극적인 최적화로 C에 근접한 성능을 내는 경우도 많다. 그러나 그 최적화가 "언제" "어떻게" 일어나는지 모르면 예측할 수 없는 성능 문제가 생긴다. 이 책이 다루는 핵심이다.

---

## JVM 구현체

JVM은 명세(specification)이고, 구현체는 여러 가지다.

- HotSpot — Oracle/OpenJDK의 기본 구현. 가장 널리 쓰인다. 이 책의 대상
- GraalVM — HotSpot 기반에 AOT 컴파일 추가
- OpenJ9 — IBM/Eclipse 계열

"JVM 튜닝"이라고 하면 대부분 HotSpot JVM 튜닝을 의미한다.
