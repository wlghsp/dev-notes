# Java 최신 기술 로드맵
## "Java 8 이후 기술을 실무에서 설명 가능한 수준으로"

문서를 만드는 게 목표가 아니다. 설명할 수 있는 수준이 목표다.

---

## 진행 원칙

실습 없이 다음 Phase로 넘어가지 않는다. "대충 알 것 같다"는 완료가 아니다 — 코드로 보여줄 수 있어야 완료다. jvm-training/db-internals와 달리 이 로드맵은 자유 학습 노트(`java/`)이므로 문서 생성 직후 Q1 테스트를 강제하지 않는다. 다만 Phase별로 직접 짧은 코드를 작성해보는 것을 권장한다.

## Phase 구성 전략

Phase 0은 기초가 흔들리는 부분(함수형 인터페이스)을 먼저 다진다. 이게 없으면 Stream, Optional, 이후 문법 대부분을 겉핥기로만 이해하게 된다. Phase 1~2는 Java 8의 나머지 핵심(Stream, Optional). Phase 3~4는 Java 9~16 사이 실무에서 자주 마주치는 것들. Phase 5~6은 Java 17/21 LTS의 핵심 문법(sealed, record, pattern matching, virtual thread). Phase 7은 최신 LTS 이후 기능 중 알아둘 만한 것.

---

## Phase 0: 함수형 인터페이스 (Java 8)

**왜 먼저인가**: Stream, Optional, 람다를 쓰는 모든 코드가 이 위에 있다. `Function<T,R>`, `Supplier<T>`, `Consumer<T>`, `Predicate<T>`가 각각 언제 쓰이는지 구분이 안 되면 이후 문법을 읽어도 "어떻게 동작하는지"만 알고 "왜 이 타입을 골랐는지"는 모른 채로 넘어가게 된다.

**다룰 내용**: `Function<T,R>`(입력 받아 출력 반환), `Supplier<T>`(입력 없이 값 생성), `Consumer<T>`(입력 받아 소비, 반환 없음), `Predicate<T>`(입력 받아 boolean 반환), `BiFunction`/`BiConsumer` 등 2-arity 변형, `@FunctionalInterface` 애너테이션의 의미, 람다식과 메서드 레퍼런스(`::`)가 함수형 인터페이스와 어떻게 연결되는지.

**실습 방향**: 각 인터페이스를 파라미터로 받는 메서드를 직접 작성해보고, 같은 로직을 일반 interface 구현체와 람다 두 가지 방식으로 짜서 코드량 차이를 비교한다.

## Phase 1: Stream API (Java 8)

**전제**: Phase 0 완료. Stream의 중간 연산(map, filter 등)은 대부분 함수형 인터페이스를 인자로 받는다.

**다룰 내용**: 중간 연산(map, filter, sorted, distinct)과 최종 연산(collect, reduce, forEach)의 구분, lazy evaluation(중간 연산은 최종 연산이 호출되기 전까지 실행되지 않음), Collectors(toList, groupingBy, joining), 병렬 스트림(parallelStream)과 언제 써야 하는지.

## Phase 2: Optional (Java 8)

**다룰 내용**: null을 대체하는 용도가 아니라 "값이 없을 수 있음을 타입으로 드러내는" 용도라는 것, `orElse`/`orElseGet`/`orElseThrow` 차이, `map`/`flatMap`으로 체이닝하는 패턴, Optional을 필드나 파라미터로 쓰면 안 되는 이유(반환 타입 전용 설계).

## Phase 3: var — 지역 변수 타입 추론 (Java 10)

**다룰 내용**: 컴파일 타임에 타입이 결정된다는 점(동적 타입이 아님), 언제 가독성을 해치는지(타입이 코드에서 드러나지 않는 경우), 언제 유용한지(제네릭이 장황한 경우).

## Phase 4: switch 표현식, 텍스트 블록 (Java 12~15)

**다룰 내용**: 기존 switch문의 fall-through 문제, `->` 화살표 문법과 값을 반환하는 switch 표현식, 여러 케이스를 묶는 문법, 텍스트 블록(`"""`)으로 멀티라인 문자열을 다루는 방법과 SQL/JSON 리터럴에 쓰이는 실무 사례.

## Phase 5: record, sealed interface, pattern matching (Java 16/17/21)

**다룰 내용**: record가 생성하는 것(생성자, accessor, equals/hashCode/toString)과 불변 데이터 클래스로 쓰는 이유. sealed interface/class는 이미 `java/sealed-interface.md`로 작성 완료 — 여기서는 record와 결합해 대수적 데이터 타입(ADT) 패턴을 만드는 법을 다룬다. `instanceof` pattern matching(캐스팅 없이 변수 바인딩), switch pattern matching과 record deconstruction(Java 21)까지 이어서 본다.

**참고**: sealed-interface.md는 이미 작성됨.

## Phase 6: Virtual Thread (Java 21)

**왜 중요한가**: 기존 Thread 모델(OS 스레드 1:1 매핑) 대비 동시성 처리 비용을 근본적으로 바꾼 변화라 실무 임팩트가 크다.

**다룰 내용**: 기존 플랫폼 스레드와의 차이(OS 스레드에 매핑되지 않고 JVM이 스케줄링), blocking I/O를 만나도 OS 스레드를 점유하지 않는 원리(carrier thread에서 unmount), 언제 유리한지(I/O 바운드 요청이 많은 서버) vs 언제 무의미한지(CPU 바운드 작업), `Executors.newVirtualThreadPerTaskExecutor()` 사용법.

## Phase 7: 최신 LTS 이후 기능 (Java 21+, 선택)

Phase 0~6을 완료한 뒤 필요에 따라 선택적으로 진행한다. 강제 순서 아님.

**후보**: Structured Concurrency(연관된 여러 비동기 작업을 하나의 단위로 묶어 관리 — Virtual Thread와 세트로 이해하는 게 자연스러움), Sequenced Collections(List/Set/Map에 첫/마지막 요소 접근 통일 인터페이스 추가), Foreign Function & Memory API(JNI 대체, native 코드 연동).

---

## 진행 현황

Phase별로 실제 작성된 파일만 여기 기록한다. 예상 파일명은 적지 않는다.

- Phase 0~7: 미작성 (`sealed-interface.md` 제외)
