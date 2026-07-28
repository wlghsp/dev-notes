# Consumer

Java 8부터 제공되는 함수형 인터페이스로, 입력 하나를 받아서 처리하지만 결과를 반환하지 않는 동작을 표현한다.

```java
public interface Consumer<T> {
    void accept(T t);
}
```

function.md에서 다룬 `Function<T,R>`이 입력을 받아 변환 결과를 내놓는다면, `Consumer<T>`는 입력을 "소비"만 하고 끝난다 — 반환 타입이 `void`다. 로깅, 출력, 필드에 값 대입처럼 "무언가를 하긴 하는데 그 결과를 다음 단계로 넘길 필요는 없는" 동작에 쓰인다.

```java
Consumer<String> printer = s -> System.out.println(s);
printer.accept("hello"); // 화면에 출력, 반환값 없음
```

## 왜 필요한가 — "실행할 동작"을 값처럼 전달

`forEach`가 대표적인 사용처다. 컬렉션의 각 원소마다 어떤 동작을 실행할지를 `Consumer`로 넘긴다.

```java
names.forEach(name -> System.out.println(name)); // forEach는 Consumer<String>을 받는다
```

`map`(Function을 받아 변환된 새 스트림을 만듦)과 다르게, `forEach`는 각 원소를 "처리"만 하고 끝나기 때문에 반환값이 없는 `Consumer`가 맞는 타입이다. 만약 원소마다 변환된 결과가 필요하다면 `Consumer`가 아니라 `Function` + `map`을 써야 한다.

## andThen — 순차 실행 합성

`Consumer`는 `Function`의 `andThen`과 비슷하지만 의미가 다르다. 값을 다음 함수로 넘기는 게 아니라, 같은 입력에 대해 동작을 순서대로 두 번 실행한다.

```java
Consumer<String> print = s -> System.out.println("print: " + s);
Consumer<String> log = s -> System.out.println("log: " + s);

Consumer<String> combined = print.andThen(log);
combined.accept("hello");
// print: hello
// log: hello
```

`Function.andThen`은 앞 결과를 뒤로 넘기지만(`g(f(x))`), `Consumer.andThen`은 반환값이 없으므로 그냥 같은 `x`를 가지고 두 동작을 차례로 실행할 뿐이다.

## 변형 — BiConsumer

입력이 두 개면 `BiConsumer<T, U>`를 쓴다. `accept(T t, U u)` 형태다. `Map.forEach((key, value) -> ...)`처럼 키와 값을 동시에 받아 처리할 때 쓰인다.

```java
map.forEach((key, value) -> System.out.println(key + "=" + value)); // BiConsumer<K,V>
```

## Function, Supplier와 구분

supplier.md, function.md에서 정리한 기준을 그대로 이어가면, 입력 없이 출력만 있으면 `Supplier<T>`, 입력과 출력이 둘 다 있으면 `Function<T,R>`, 입력만 있고 출력이 없으면 `Consumer<T>`다.
