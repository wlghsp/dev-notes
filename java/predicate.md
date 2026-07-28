# Predicate

Java 8부터 제공되는 함수형 인터페이스로, 입력 하나를 받아 boolean을 반환하는 조건 판별 동작을 표현한다.

```java
public interface Predicate<T> {
    boolean test(T t);
}
```

function.md의 `Function<T,R>`이 출력 타입을 자유롭게 정할 수 있는 것과 달리, `Predicate<T>`는 출력이 항상 boolean으로 고정된 특화된 형태다. `Function<T, Boolean>`으로도 같은 걸 표현할 수 있지만, "조건을 검사한다"는 의도를 타입 이름으로 명확히 드러내고 아래에 나오는 조건 조합용 메서드(and/or/negate)까지 제공한다는 점에서 `Predicate`를 따로 쓴다.

```java
Predicate<String> isEmpty = s -> s.isEmpty();
isEmpty.test(""); // true
```

## 왜 필요한가 — "조건"을 값처럼 전달

`Stream.filter`가 대표적인 사용처다. 어떤 원소를 남길지 조건을 `Predicate`로 넘긴다.

```java
List<String> longNames = names.stream()
    .filter(name -> name.length() > 3) // filter는 Predicate<String>을 받는다
    .toList();
```

조건 자체가 재사용 가능한 값이 되므로, 여러 곳에서 같은 조건을 변수로 선언해 공유하거나, 조건끼리 조합해서 새로운 조건을 만들 수 있다.

## and, or, negate — 조건 조합

`Predicate`는 논리 연산자에 대응하는 기본 메서드를 제공해서 여러 조건을 코드로 조립할 수 있다.

```java
Predicate<String> isNotEmpty = s -> !s.isEmpty();
Predicate<String> isShort = s -> s.length() < 5;

Predicate<String> combined = isNotEmpty.and(isShort); // 둘 다 만족
Predicate<String> either = isNotEmpty.or(isShort);    // 하나만 만족해도 됨
Predicate<String> opposite = isNotEmpty.negate();     // 조건 반전 (isEmpty와 동일)
```

`and`/`or`는 단축 평가(short-circuit)로 동작한다. `and`에서 앞 조건이 false면 뒤 조건은 아예 평가하지 않고, `or`에서 앞 조건이 true면 마찬가지로 뒤는 평가하지 않는다 — `&&`, `||` 연산자와 같은 동작이다.

## 변형 — BiPredicate

입력이 두 개면 `BiPredicate<T, U>`를 쓴다. `test(T t, U u)` 형태다. 두 값을 비교해서 조건을 판별할 때 쓰인다.

```java
BiPredicate<String, Integer> lengthMatches = (s, len) -> s.length() == len;
```

## 네 가지 함수형 인터페이스 정리

Phase 0에서 다룬 네 가지를 입력·출력 유무로 정리하면 다음과 같다. 입력 없이 출력만 있으면 `Supplier<T>`(supplier.md), 입력과 출력이 둘 다 있으면 `Function<T,R>`(function.md), 입력만 있고 출력이 없으면 `Consumer<T>`(consumer.md), 입력을 받아 boolean만 반환하면 `Predicate<T>`다. `Predicate`도 사실 출력이 있는 `Function<T, Boolean>`의 특화 형태지만, 조건 판별이라는 용도와 and/or/negate 조합 기능 때문에 별도 타입으로 존재한다.
