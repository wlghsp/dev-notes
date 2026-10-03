# Mockito 스텁 문법: mock, spy, given/will 순서

## 왜 헷갈리나

**스텁을 거는 문법이 두 가지고, 어떤 걸 써야 하는지가 대상(mock/spy)과 메서드(void/non-void)에 따라 갈리기 때문.** 둘이 같은 일을 하는 것처럼 보이다가 어느 순간 컴파일 에러나 이상한 동작으로 드러난다.

## stub과 stubbing

**stub은 "이 메서드가 호출되면 이렇게 동작해라"라고 미리 정해둔 가짜 동작이고, stubbing은 그 동작을 지정하는 행위다.** "스텁을 건다", "메서드를 스텁한다"는 모두 같은 말이다.

```java
given(repo.find(1L)).willReturn(user);   // 이 줄이 stubbing, 결과로 붙은 "user를 돌려준다"가 stub
```

스텁을 안 걸면 mock은 기본값(`null` 등)을 돌려준다.

원래 테스트 더블 용어로는 stub이 **입력 공급**(값을 돌려주거나 예외를 던짐), mock이 **호출 검증**(`verify`)이다. Mockito는 `mock` 객체 하나로 둘 다 하기 때문에 실무에서는 "스텁 건다"를 "mock의 동작을 지정한다"는 뜻으로 쓴다.

```java
given(repo.find(1L)).willReturn(user);   // 스텁: 입력 준비
service.process(1L);
then(repo).should().find(1L);            // 검증: 호출 확인
```

## mock vs spy

**mock** — 껍데기만 있는 가짜 객체. 스텁하지 않은 메서드는 아무것도 안 하고 기본값을 돌려준다.

**spy** — 진짜 객체를 감싼 것. 스텁하지 않은 메서드는 **실제 코드가 그대로 실행된다.** 일부 메서드만 가짜로 바꾸고 싶을 때 쓴다.

- 스텁 안 한 메서드: mock은 기본값(`null`, `0`, `false`, 빈 컬렉션, `Optional.empty()`)을 돌려주고, spy는 실제 메서드를 실행한다.
- 실제 객체: mock은 필요 없고, spy는 필요하다(`spy(new Foo())`).
- 주 용도: mock은 협력 객체 대체, spy는 테스트 대상 일부만 막기.

```java
List<String> mock = mock(ArrayList.class);
mock.add("a");
mock.size();        // 0 — add가 실제로 동작하지 않았다

List<String> spy = spy(new ArrayList<>());
spy.add("a");
spy.size();         // 1 — 진짜 ArrayList
```

## 스텁 문법 두 가지

**1. when 스타일 — 값을 먼저 쓴다**

```java
given(mock.find(1L)).willReturn(user);
given(mock.find(1L)).willThrow(new RuntimeException());
```

**2. do 스타일 — 동작을 먼저 쓴다**

```java
willReturn(user).given(mock).find(1L);
willThrow(new RuntimeException()).given(mock).find(1L);
willDoNothing().given(mock).delete(1L);
```

BDDMockito 이름과 Mockito 원래 이름은 1:1 대응이다.

- `given(x)` = `when(x)`
- `willReturn` = `doReturn`
- `willThrow` = `doThrow`
- `willAnswer` = `doAnswer`
- `willDoNothing` = `doNothing`

## 핵심: given(mock.find(1L))은 호출이다

**`given(...)` 괄호 안의 `mock.find(1L)`은 스텁 설정 시점에 실제로 한 번 호출된다.** Mockito는 "마지막으로 호출된 메서드"를 기억했다가 거기에 동작을 붙이는 방식이기 때문이다.

이 사실에서 아래 세 가지 제약이 전부 나온다.

## do 스타일을 써야 하는 경우

**1. void 메서드**

void는 반환값이 없어서 `given()`의 인자로 넣을 수 없다. 컴파일이 안 된다.

```java
given(service.detach(id)).willThrow(ex);       // 컴파일 에러
willThrow(ex).given(service).detach(id);       // 정상
```

**2. spy**

`given(spy.method())`은 괄호 안에서 **실제 메서드가 실행된다.** DB를 건드리거나 예외를 던지는 메서드면 스텁을 걸기도 전에 터진다.

```java
given(spy.save(entity)).willReturn(entity);    // save()가 진짜로 실행됨
willReturn(entity).given(spy).save(entity);    // 실행 없이 설정만
```

**3. 이미 예외를 던지도록 스텁한 메서드를 다시 스텁할 때**

mock에서도 같은 이유로 터진다. 첫 스텁이 `willThrow`면 두 번째 `given(mock.find())`의 괄호 안 호출에서 예외가 난다.

```java
given(mock.find(1L)).willThrow(ex);
given(mock.find(1L)).willReturn(user);         // 여기서 ex가 터진다
willReturn(user).given(mock).find(1L);         // 정상
```

## 그 외에는 when 스타일이 낫다

**non-void이고 일반 mock이면 `given(...).willReturn(...)`이 읽기 쉽다.** 왼쪽에서 오른쪽으로 "이걸 호출하면 → 이걸 돌려준다"로 읽히고, 반환 타입도 컴파일러가 검사해준다. do 스타일은 타입 검사가 약하다(`willReturn`의 인자 타입이 메서드 반환 타입과 안 맞아도 컴파일은 되고 런타임에 터진다).

## 연속 호출 스텁

**호출 순서대로 다른 동작을 주려면 체이닝한다.** 재시도 로직 테스트에서 자주 쓴다.

```java
// 첫 호출은 예외, 두 번째부터는 아무것도 안 함 (void)
willThrow(new RuntimeException("실패"))
        .willDoNothing()
        .given(service).detach(id);

// non-void
given(mock.find(1L))
        .willThrow(new RuntimeException())
        .willReturn(user);
```

마지막에 지정한 동작이 이후 호출에도 계속 적용된다.

## willReturn vs willAnswer

**`willReturn`은 값을 고정하고, `willAnswer`는 호출될 때마다 람다로 계산한다.**

```java
given(repo.save(any())).willAnswer(inv -> inv.getArgument(0));   // 받은 걸 그대로 반환
```

`save()`에 어떤 객체가 들어올지 테스트 시점엔 모를 때(서비스 안에서 만든 엔티티), 인자를 그대로 돌려주는 용도로 쓴다.

## spy를 쓸 때 주의

**하나, spy는 원본을 복사한다.** `spy(obj)` 이후 원본 `obj`를 바꿔도 spy에는 반영되지 않는다. 테스트에서는 spy만 쓴다.

**둘, final 클래스/메서드, private 메서드는 스텁할 수 없다.** (mockito-inline이 기본인 최신 Mockito는 final은 가능하지만 private은 여전히 불가)

**셋, spy가 필요해졌다는 건 설계 신호일 수 있다.** 테스트 대상 클래스의 일부 메서드를 막아야 한다면 그 부분을 별도 협력 객체로 분리해서 mock으로 대체하는 쪽이 대개 더 깔끔하다.

## Spring에서

- 순수 Mockito mock — `@Mock`
- 순수 Mockito spy — `@Spy`
- 스프링 컨텍스트의 빈을 mock으로 교체 — `@MockBean` → Boot 3.4부터 `@MockitoBean`
- 스프링 컨텍스트의 빈을 spy로 교체 — `@SpyBean` → Boot 3.4부터 `@MockitoSpyBean`

`@MockBean`/`@SpyBean`은 Boot 3.4부터 deprecated이고 스프링 프레임워크의 `@MockitoBean`/`@MockitoSpyBean`으로 대체됐다. 단위 테스트(`@ExtendWith(MockitoExtension.class)`)에서는 스프링 컨텍스트가 없으니 `@Mock`/`@Spy`를 쓴다.

## 한눈에

- non-void, mock — `given(mock.m()).willReturn(x)`
- void 메서드 — `willThrow(ex).given(mock).m()`
- spy — `willReturn(x).given(spy).m()`
- 예외 스텁 덮어쓰기 — do 스타일
- 호출마다 다른 동작 — `.willThrow(a).willReturn(b)` 체이닝
- 받은 인자를 그대로 반환 — `willAnswer(inv -> inv.getArgument(0))`
