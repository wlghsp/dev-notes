# AOP / Aspect / Advice 용어

AOP(Aspect-Oriented Programming)와 그 하위 용어(Aspect, Advice, Pointcut, Joinpoint)가 왜 이런 이름으로 불리는지 정리한다. 동작 방식은 spring-aop-proxy.md에서 다루고, 여기서는 용어 자체의 유래에 집중한다.

## 왜 "Aspect(관점)"인가

객체지향 프로그래밍은 기능을 클래스 단위로 나눈다. `TodoService`는 Todo 조회 로직을, `UserService`는 User 로직을 담당하는 식이다. 그런데 로깅, 트랜잭션, 호출 횟수 기록 같은 관심사는 이 클래스 구분과 다른 축을 따라 흩어져 있다 — `TodoService.findById`에도, `UserService.findById`에도 똑같이 필요할 수 있다.

이렇게 하나의 클래스 구조로는 깔끔하게 나눌 수 없고 여러 모듈에 걸쳐 반복되는 관심사를 cross-cutting concern(횡단 관심사)이라 부른다. 이런 관심사를 "클래스"가 아니라 별도의 "관점(aspect)"으로 바라보고 하나의 모듈로 뽑아내자는 게 AOP의 핵심 아이디어다. `CallCountingAspect`라는 이름 자체가 "호출 횟수 기록이라는 관점을 담은 모듈"이라는 뜻이다.

## 왜 "Advice(조언)"인가

Aspect 모듈 안에서 실제로 실행되는 코드 조각(예: `callCounter.increment()`)을 advice라 부른다. "조언"이라는 단어가 붙은 이유는, 원래 메서드(target)의 실행 흐름에 "이것도 같이 해달라"고 끼워 넣는 부가 지시라는 의미에서다 — 원래 로직을 대체하는 게 아니라, 그 실행에 덧붙이는 지시라는 뉘앙스를 담고 있다.

`@Before`, `@After`, `@Around` 같은 어노테이션은 이 advice를 "언제" 실행할지 구분하는 것이다. `@Around`는 대상 메서드 실행 전후를 모두 감싸는 advice이고, `joinPoint.proceed()` 호출이 "원래 메서드를 실제로 실행해라"는 지점이 된다.

## Pointcut과 Joinpoint

- **Joinpoint(조인포인트)**: advice를 끼워 넣을 수 있는 실행 시점 후보 전체 — 메서드 호출, 생성자 호출 등. Spring AOP는 이 중 메서드 실행만 지원한다.
- **Pointcut(포인트컷)**: 수많은 joinpoint 중 "이 aspect를 실제로 어디에 적용할지" 골라내는 표현식. `execution(* co.dingcodingco.challenge.todo.TodoService.findById(..))`처럼 조건을 적으면, 그 조건에 맞는 joinpoint(여기서는 `findById` 메서드 호출)에만 advice가 연결된다.

정리하면 "Aspect가 Pointcut으로 지정된 Joinpoint에 Advice를 끼워 넣는다"는 한 문장에 이 네 용어가 다 들어있다.

## 왜 이렇게 나눠 부르는가

각 용어가 답하는 질문이 다르기 때문이다.

- Aspect — 무엇에 대한 관심사인가 (모듈)
- Advice — 무엇을 할 것인가 (실행할 코드, 그리고 언제 실행할지)
- Pointcut — 어디에 적용할 것인가 (대상 선택 규칙)
- Joinpoint — 적용될 수 있는 지점은 무엇인가 (실행 시점 후보)

이 네 가지를 분리해두면, "무엇을 할지"(advice)와 "어디에 적용할지"(pointcut)를 독립적으로 바꿀 수 있다. 같은 `CallCountingAspect`의 advice 코드를 그대로 두고 포인트컷만 `TodoService.create`로 바꾸면 적용 대상이 바뀌는 식이다 — 관심사(무엇을 할지)와 적용 범위(어디에 할지)를 분리하는 게 AOP 설계의 핵심이라, 각 개념에 별도 이름이 붙어 있다.

참고: spring-aop-proxy.md
참고: self-invocation-and-spring-aop-proxy.md
