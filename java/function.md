# Function

Java 8부터 제공되는 함수형 인터페이스로, 입력 하나를 받아 출력 하나를 반환하는 동작을 표현한다.

```java
public interface Function<T, R> {
    R apply(T t);
}
```

`T`는 입력 타입, `R`은 출력 타입이다. supplier.md에서 다뤘듯 `Supplier<T>`는 입력이 없고, `Function<T,R>`은 입력을 받아 변환한 결과를 내놓는다는 점이 다르다.

```java
Function<String, Integer> length = s -> s.length();
int result = length.apply("hello"); // 5
```

## 왜 필요한가 — "변환 로직"을 값처럼 전달

`Function`이 없다면 문자열을 정수로 바꾸는 로직, 대문자로 바꾸는 로직마다 별도 메서드를 만들거나 인터페이스를 새로 정의해야 한다. `Function<T,R>`은 "입력 하나 받아서 출력 하나 내는" 모든 변환 로직에 재사용 가능한 공통 타입을 제공한다. 그래서 Stream의 `map`처럼 "각 원소를 어떻게 변환할지"를 메서드 인자로 받아야 하는 자리에 표준적으로 쓰인다.

```java
List<Integer> lengths = names.stream()
    .map(name -> name.length()) // map은 Function<String, Integer>를 받는다
    .toList();
```

## compose와 andThen — 함수 합성

`Function`은 여러 개를 이어붙이는 기본 메서드를 제공한다.

`andThen`은 현재 함수를 먼저 실행하고, 그 결과를 다음 함수에 넘긴다.

```java
Function<Integer, Integer> plusOne = x -> x + 1;
Function<Integer, Integer> times2 = x -> x * 2;

Function<Integer, Integer> combined = plusOne.andThen(times2);
combined.apply(3); // (3+1)*2 = 8
```

`compose`는 순서가 반대다. 인자로 받은 함수를 먼저 실행하고, 그 결과를 현재 함수에 넘긴다.

```java
Function<Integer, Integer> combined2 = plusOne.compose(times2);
combined2.apply(3); // (3*2)+1 = 7
```

`f.andThen(g)`는 `g(f(x))`, `f.compose(g)`는 `f(g(x))`로 기억하면 순서가 헷갈리지 않는다.

## 변형 — BiFunction, UnaryOperator

입력이 두 개면 `BiFunction<T, U, R>`을 쓴다. `apply(T t, U u)` 형태로 인자가 하나 늘어난다.

입력과 출력 타입이 같은 경우(`Function<T, T>`)에는 `UnaryOperator<T>`라는 특화된 타입이 따로 있다. 예를 들어 문자열을 받아 문자열을 반환하는 변환은 `Function<String, String>`으로도 쓸 수 있지만, 관례적으로 입출력 타입이 같음을 드러내고 싶을 때 `UnaryOperator<String>`을 쓴다.

## Supplier, Consumer와 구분

입력과 출력 유무로 셋을 구분하면 명확하다. 입력 없이 출력만 있으면 `Supplier<T>`, 입력만 있고 출력이 없으면 `Consumer<T>`, 입력과 출력이 둘 다 있으면 `Function<T,R>`이다.
