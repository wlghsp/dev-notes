# 카카오톡 Java 서버 리팩토링 — 쉽게 이해하기

> 원문: [카카오 테크 블로그 — Java App Server 리팩토링 후기](https://tech.kakao.com/posts/566)
> 카카오톡 메시징 서버를 실제 운영 중에 성능 저하 없이 리팩토링한 경험을 공유한 글.

---

## 왜 리팩토링이 필요했나?

오랫동안 운영된 서버 코드에는 공통적으로 3가지 문제가 생긴다.

1. **Context 클래스 남용** — 상태를 담는 객체가 전역 변수처럼 퍼져나감
2. **서비스 간 순환 의존성** — A가 B를 쓰고, B가 A를 쓰고, C가 A와 B를 모두 쓰는 상황
3. **복잡도 폭발** — 중첩된 조건문, 분기가 너무 많아 코드 흐름 파악 불가

이 글은 이 세 가지를 어떻게 해결했는지 다룬다.

---

## 1. 가변 Context 클래스 — 신중하게 써라

### Context 클래스란?

"지금 상황"을 담아두는 객체. 예를 들어 카카오톡에서는 아래 같은 것들이 Context가 된다.

- 로그인 상태
- 채팅방 참여 상태
- 친구 관계 상태

코드로 보면 이렇게 생겼다.

```java
class CarContext {
    private Car car;
    private List<People> passengers;
    private List<People> visitors;
    private SomethingBig somethingBig;
    // ... 수십 개의 필드
    // ... 수십 개의 get/set 메서드
}
```

### 뭐가 문제인가?

Context가 여러 함수로 전달되다 보면 **전역 변수처럼 작동**한다.

```
함수A: carContext.setVisitors(...)    // 여기서 set
함수B: carContext.getVisitors()       // 저기서 get
함수C: carContext.setCar(...)         // 또 다른 곳에서 set
```

어디서 무엇을 set했는지 추적하기 어렵고, 버그가 생기면 원인 찾기가 극도로 힘들어진다.

---

### 해결책 1: Context 대신 반환값으로 돌려줘라

**Before**
```java
Result createCar(CarContext carContext, Something A, Something B) {
    ...
    carContext.setCar(new Car()); // Context에 집어넣음
    ...
    return new Result(someValue);
}
```

**After**
```java
Pair<Result, Car> createCar(Something A, Something B) {
    ...
    return new Pair<>(new Result(someValue), new Car()); // 반환값으로 돌려줌
}
```

Context에 의존하지 않고, 함수가 필요한 것을 직접 반환한다. **함수의 의도가 명확해지고 결합도가 낮아진다.**

---

### 해결책 2: Context 전체 말고 필요한 값만 받아라

**Before**
```java
void doSomethingForPassengers(CarContext carContext, Something A, Something B) {
    List<People> passengers = carContext.getPassengers(); // Context에서 꺼냄
    SomeValue someValue = carContext.getSomeValue();      // Context에서 꺼냄
    ...
}
```

**After**
```java
void doSomethingForPassengers(List<People> passengers, SomeValue someValue,
        Something A, Something B) {
    // 필요한 값을 파라미터로 직접 받음
    ...
}
```

"이 함수는 무엇이 필요한가"가 시그니처에 명확하게 드러난다. Context를 통째로 받으면 실제로 무엇을 쓰는지 함수 내부를 열어봐야만 알 수 있다.

---

### 해결책 3: 캐싱이 목적이라면 루프 밖으로 꺼내라

Context에 값을 집어넣는 이유 중 하나가 "루프 안에서 매번 DB 조회하지 않으려고"인 경우가 많다.

**Before**
```java
void prepareVisitors(CarContext carContext, Condition condition) {
    if (carContext.getVisitors() == null) {
        carContext.setVisitors(readFromDB(condition)); // Context에 캐싱
    }
}

void doSomethingForVisitors(CarContext carContext, List<Something> somethings,
        Condition condition) {
    for (Something something : somethings) {
        doSomeProcess(carContext, condition); // Context를 계속 들고 다님
        List<People> visitors = carContext.getVisitors();
        ...
    }
}
```

**After**
```java
void doSomethingForVisitors(List<Something> somethings, Condition condition) {
    List<People> visitors = readFromDB(condition); // 루프 시작 전에 한 번만 조회
    for (Something something : somethings) {
        doSomeProcess(visitors); // 필요한 값만 전달
        ...
    }
}
```

Context가 없어도 캐싱이 된다. Context는 불필요한 필드와 get/set 메서드가 줄어들어 간결해진다.

---

### Context 클래스 사용 원칙 요약

| 하지 말 것 | 대신 할 것 |
|---|---|
| Context를 전역 변수처럼 사용 | 필요한 값을 파라미터로 명시 |
| Context를 함수에 통째로 전달 | 반환값이나 직접 파라미터로 처리 |
| 캐싱용으로 Context에 저장 | 루프 밖으로 추출 |

**결과**: 카카오톡 서버에서 10개의 Context 클래스를 3개로 줄이고, 수백 줄의 코드를 삭제했다.

---

## 2. 고차 함수로 의존성 줄이기

### 순환 의존성이란?

```
ServiceA → ServiceB 사용
ServiceB → ServiceC 사용
ServiceC → ServiceA 사용 (순환!)
```

이렇게 되면 ServiceA를 테스트하려면 ServiceB가 필요하고, ServiceB를 만들려면 ServiceC가 필요하고, ServiceC를 만들려면 ServiceA가 필요하다. **셋 다 동시에 만들어야 하는 상황**이 된다.

스프링에서는 이걸 억지로 허용하는 설정이 있지만 이건 임시방편이다.

```yaml
spring.main.allow-circular-references=true  # 이건 해결책이 아님
```

---

### 왜 생기나?

스프링의 의존성 주입(`@Resource`, `@Autowired`)을 남용하면 서비스 간 관계가 점점 복잡해진다.

```java
@Service
public class ServiceA {
    @Resource
    ServiceB serviceB; // ServiceA가 ServiceB를 직접 알고 있음

    public void methodA(Integer param) {
        serviceB.methodB(param); // ServiceB 메서드를 직접 호출
    }
}
```

`ServiceA`가 `ServiceB`라는 클래스 전체를 알아야 한다. ServiceB의 구현이 바뀌면 ServiceA도 영향을 받는다.

---

### 해결책: 클래스가 아닌 함수를 주입해라

**Step 1: ServiceA를 수정해서 ServiceB 대신 함수를 받도록**

```java
@Service
public class ServiceA {
    // ServiceB 객체를 받지 않음
    public void methodA(Integer param, Function<Integer, Integer> methodB) {
        // "Integer를 넣으면 Integer를 돌려주는 함수"만 알면 됨
        System.out.println(methodB.apply(param));
    }
}
```

**Step 2: Handler에서 조립**

```java
@Component
public class Handler {
    private final ServiceA serviceA;
    private final ServiceB serviceB;

    public Handler(ServiceA serviceA, ServiceB serviceB) {
        this.serviceA = serviceA;
        this.serviceB = serviceB;
    }

    public void execute() {
        serviceA.methodA(2, serviceB::methodB); // 메서드 참조로 함수를 넘김
    }
}
```

`serviceB::methodB`는 "ServiceB의 methodB 메서드를 함수로 표현한 것"이다.

---

### 비유로 이해하기

**Before (객체 의존)**

> "너 ServiceB 전체를 내 방으로 데려와. 내가 거기서 methodB를 골라 쓸게."

**After (함수 의존)**

> "methodB처럼 동작하는 함수 하나만 줘. 그게 어디서 왔는지 나는 몰라도 돼."

ServiceA는 ServiceB가 어떻게 생겼는지, 어떤 다른 메서드가 있는지 전혀 알 필요가 없어진다.

---

### 테스트가 얼마나 쉬워지나?

**Before (스프링/Mockito 없이는 불가)**

```java
// @SpringBootTest 또는 @Mock/@InjectMocks 필요
// 스프링 컨테이너를 띄워야 테스트 가능
```

**After (순수 Java로 테스트 가능)**

```java
class SampleUnitTests {
    @Test
    public void testServiceB() {
        ServiceB serviceB = new ServiceB(); // 그냥 new로 만들면 됨
        Integer result = serviceB.methodB(2, () -> 10, (a, b) -> a + b.getAsInt());
        assertThat(result, equalTo(12));
    }

    @Test
    public void testHandler() {
        // 모든 서비스를 그냥 new로 만들어서 조립
        Handler handler = new Handler(new ServiceA(), new ServiceB(), new ServiceC());
        handler.execute(1);
    }
}
```

Mockito도, `@SpringBootTest`도 필요 없다. 그냥 `new`로 만들어서 테스트할 수 있다.

---

### 성능 문제는 없나?

함수를 파라미터로 넘기면 약간의 성능 저하가 생긴다. 하지만 JVM의 JIT 컴파일러가 최적화하면 그 차이는 무시할 수준이다.

카카오톡 라이브 서비스에 반영했을 때 성능 저하는 없었다.

> **결론: 유지보수성 향상 >> 미세한 성능 차이**

---

### 고차 함수 의존성 요약

| | 객체 의존 | 함수 의존 |
|---|---|---|
| ServiceA가 알아야 하는 것 | ServiceB 클래스 전체 | 함수 시그니처 하나 |
| 순환 의존성 | 발생 가능 | 없음 |
| 테스트 | 스프링/Mockito 필요 | `new`로 바로 가능 |
| 결합도 | 높음 | 낮음 |

---

## 3. 코드 복잡도 수치로 줄이기

### "복잡하다"를 어떻게 수치로 표현하나?

두 가지 지표를 쓴다.

---

### Cyclomatic Complexity (CC)

**분기점의 수**를 센다. 분기점이 많을수록 점수가 높고, 테스트해야 할 경우의 수가 많다는 뜻이다.

```
기본값: 1
if 하나 추가: +1
for 하나 추가: +1
조건식 안의 || 또는 && 추가: +1
```

예시:
```java
void example(boolean isA, boolean isB) {
    if (isA || isB) {  // +2 (if +1, || +1)
        for (int i = 0; i < 10; i++) {  // +1
            ...
        }
    }
}
// CC = 1 + 2 + 1 = 4
```

---

### NPath Complexity (NPath)

**가능한 실행 경로의 수**. 분기마다 곱해서 계산하기 때문에 CC보다 훨씬 빠르게 커진다.

```
if: ×2
if/else if: ×3
for: ×2
```

예시:
```java
void example() {
    for (...) {      // ×2
        for (...) {  // ×2
            if (...) {  // ×2 (for 안에서 조건이 있어 합산)
            }
        }
    }
}
// NPath = 2 × 2 × (2+1) ≈ 12
```

CC와 NPath를 같이 보면 "이 함수가 얼마나 복잡한가"를 객관적으로 판단할 수 있다.

> **도구**: IntelliJ Plugin — "Complexity reducer"

---

### 복잡한 함수 리팩토링 예시

**원본 (CC: 6, NPath: 8)**

```java
public Data buildData(boolean isConditionA, boolean isConditionB,
        boolean isConditionC, String extraCondition) {
    int someValue;
    if (isConditionA) {
        someValue = 10;
    } else {
        if (extraCondition.equals("ForceB") || isConditionB) {
            someValue = 20;
        } else {
            if (isConditionC) {
                someValue = 30;
            } else {
                someValue = 40;
            }
        }
    }

    Data data = new Data();
    data.setA(someValue + 1);
    if (someValue == 30) {
        data.setB(someValue + 2);
    } else {
        data.setB(someValue + 4);
    }
    data.setC(someValue + 3);
    return data;
}
```

이 함수는 두 가지 일을 한다.
1. `someValue` 계산
2. `Data` 객체 생성

**하나의 함수가 두 가지 책임을 가지고 있으므로 나눈다.**

---

### 리팩토링 Step 1: 함수 추출

```java
// someValue 계산만 담당
private int getSomeValue(boolean isConditionA, boolean isConditionB,
        boolean isConditionC, String extraCondition) {
    if (isConditionA) {
        return 10;
    } else if (extraCondition.equals("ForceB") || isConditionB) {
        return 20;
    } else if (isConditionC) {
        return 30;
    }
    return 40;
}

// Data 객체 생성만 담당
private Data makeData(int base) {
    Data data = new Data();
    data.setA(base + 1);
    if (base == 30) {
        data.setB(base + 2);
    } else {
        data.setB(base + 4);
    }
    data.setC(base + 3);
    return data;
}

// 조립만 담당 (CC: 1, NPath: 1)
public Data buildData(boolean isConditionA, boolean isConditionB,
        boolean isConditionC, String extraCondition) {
    return makeData(getSomeValue(isConditionA, isConditionB, isConditionC, extraCondition));
}
```

---

### 리팩토링 Step 2: 중첩 조건을 보호 구문으로 변환

중첩된 `if-else` 대신 `else if`를 사용해 가독성을 높인다.

**Before (중첩 if-else)**
```java
if (isConditionA) {
    return 10;
} else {
    if (isConditionB) {
        return 20;
    } else {
        if (isConditionC) {
            return 30;
        } else {
            return 40;
        }
    }
}
```

**After (보호 구문)**
```java
if (isConditionA) {
    return 10;
} else if (isConditionB) {
    return 20;
} else if (isConditionC) {
    return 30;
}
return 40;
```

코드가 오른쪽으로 들여쓰기되는 "화살표 패턴"이 사라지고 읽기 쉬워진다.

> **주의**: 중첩 `if-else`를 단순 `if`로만 바꾸면 NPath가 일시적으로 증가한다. `else if`로 연결해야 NPath가 낮아진다.

---

### 복잡도 원칙 요약

**"함수가 하나의 책임만 가지도록 한다면, 각 함수의 복잡도는 낮아지고 코드 추가도 간단해진다."**

| 리팩토링 기법 | 효과 |
|---|---|
| 함수 추출 | CC/NPath 분산, 각 함수 책임 명확화 |
| 중첩 조건 → else if | 화살표 패턴 제거, NPath 감소 |
| 조기 반환(early return) | 불필요한 중첩 제거 |

---

## 전체 결과

카카오톡 메시징 서버에 이 세 가지를 적용한 결과:

| 항목 | Before | After |
|---|---|---|
| Context 클래스 수 | 10개 | 3개 |
| 코드 라인 | 기준 | 수백 줄 삭제 |
| 단위 테스트 | 스프링 컨테이너 필요 | `new`로 바로 가능 |
| 운영 성능 | 기준 | 저하 없음 |
| 순환 의존성 | 존재 | 제거 |

---

## 핵심 3줄 요약

1. **Context는 전역 변수가 아니다** — 필요한 값만 파라미터로 받고 반환값으로 돌려줘라.
2. **클래스가 아닌 함수에 의존해라** — 고차 함수로 순환 의존성을 끊고 테스트를 쉽게 만들어라.
3. **복잡도는 수치로 측정해라** — CC와 NPath로 객관화하고, 함수 추출과 보호 구문으로 낮춰라.
