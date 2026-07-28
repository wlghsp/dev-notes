# Supplier

Java 8부터 제공되는 함수형 인터페이스로, 입력은 받지 않고 값을 반환(생성)하는 동작을 표현한다.

```java
public interface Supplier<T> {
    T get();
}
```

`Function<T,R>`은 입력을 받아 출력을 만들고, `Consumer<T>`는 입력만 받고 반환이 없는 것과 비교하면, `Supplier<T>`는 입력이 아예 없다는 게 특징이다. "값을 어떻게 만들지"에 대한 로직만 담고 있고, 그 로직을 실제로 실행할지 말지는 호출하는 쪽이 `get()`을 부르는 시점에 결정한다.

## 왜 필요한가 — 값 대신 "값을 만드는 방법"을 전달

일반 메서드는 인자로 값을 받으면 호출 시점에 그 값이 즉시 계산되어 전달된다.

```java
void log(String message) { ... }
log(buildExpensiveMessage()); // 이 메서드가 항상 호출됨, log 내부에서 안 써도 이미 계산됨
```

`Supplier`를 인자로 받으면 "값을 만드는 방법"만 전달하고, 실제 계산은 필요한 시점에 `get()`을 호출할 때 일어난다.

```java
void log(Supplier<String> messageSupplier) {
    if (logLevelEnabled) {
        log(messageSupplier.get()); // 필요할 때만 계산됨
    }
}
log(() -> buildExpensiveMessage());
```

로그 레벨이 꺼져 있으면 `buildExpensiveMessage()`는 아예 실행되지 않는다. 이렇게 "계산을 미루는" 성격을 지연 평가(lazy evaluation)라고 부른다.

## 실무에서 자주 보이는 자리

`Optional.orElseGet(Supplier<T>)`가 대표적이다. `orElse(value)`는 값이 없을 때 쓸 기본값을 인자로 받는데, 이 인자는 Optional에 값이 있든 없든 항상 즉시 계산된다. `orElseGet(supplier)`는 Optional이 비어있을 때만 `supplier.get()`을 호출하므로, 기본값을 만드는 데 비용이 크다면(DB 조회, 객체 생성 등) `orElseGet`을 써야 불필요한 계산을 피할 수 있다.

객체를 새로 만드는 팩토리 역할로도 쓰인다. `Stream.generate(Supplier<T>)`는 `get()`을 반복 호출해서 무한 스트림을 만들고, 테스트 코드에서 매번 새 인스턴스가 필요할 때 `Supplier<TestObject>`를 넘겨 재사용하는 패턴도 흔하다.

## Function과 헷갈릴 때 구분법

인자를 받는지만 보면 된다. 인자 없이 값만 나오면 `Supplier`, 인자를 받아 값으로 변환하면 `Function`이다. `Function<Void, T>`로 억지로 흉내낼 수도 있지만, 실제로 `Void`를 인자로 넘겨야 해서 어색하다 — 애초에 입력이 없는 경우를 위해 `Supplier`가 따로 존재하는 이유다.
