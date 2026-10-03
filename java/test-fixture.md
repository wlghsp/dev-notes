# 테스트 픽스처 작성 가이드

## 픽스처가 뭔가

**테스트마다 반복해서 만드는 가짜 샘플 데이터를 한 곳에 모아둔 것.** 테스트 코드에 매번 값을 새로 타이핑하지 않고, 이 클래스에서 가져다 쓴다.

```java
public final class UserFixture {
    public static final String EMAIL = "user@test.com";

    private UserFixture() {}

    public static User basicUser() {
        return User.create(EMAIL, "홍길동");
    }
}
```

## 왜 필요한가

**같은 데이터를 여러 테스트가 반복해서 쓰기 때문.** 값을 여기저기 직접 타이핑하면 오타도 나고, 나중에 값을 바꿀 때 다 찾아 고쳐야 한다. 한 곳에 정의해두면 검증 코드(`assertThat`)에서도 같은 상수를 그대로 참조할 수 있다.

## 구성 3가지

**1. 상수** — 반복해서 쓸 값들. 이름을 붙여서 재사용한다.

```java
public static final String PROVIDER_ID = "provider-1";
```

**2. 생성자 private + 클래스 final** — 인스턴스화 방지. 정적 메서드만 쓸 거라서.

```java
private DummyPortFixture() {}
```

**3. 정적 팩토리 메서드** — "어떤 상태의 객체"를 만들지 이름으로 드러낸다.

```java
public static DummyPortAllocation activeAllocation() { ... }
public static DummyPortAllocation creatingAllocation() { ... }
```

## 이름 짓는 법

**메서드 이름이 곧 "어떤 상황인지" 설명이어야 한다.** 읽는 사람이 메서드 본문을 안 봐도 뭘 리턴하는지 알아야 한다.

- `basicUser()` — 평범한 케이스
- `activeAllocation()` — 이미 완료(ACTIVE) 상태
- `creatingAllocation()` — 방금 생성된(CREATING) 상태
- `expiredToken()` — 만료된 토큰

## 상태가 있는 객체는 "쌓아서" 만든다

**실제 상태 전이 메서드를 그대로 순서대로 호출해서 원하는 상태를 만든다.** 필드를 강제로 바꾸지 않는다 — 그러면 실제 코드가 그 상태에 도달하는 경로를 안 타게 된다.

```java
public static DummyPortAllocation activeAllocation() {
    DummyPortAllocation allocation = creatingAllocation();
    allocation.markPortCreated(PORT_ID);
    allocation.markAttached();
    allocation.markActive();
    return allocation;
}
```

## 흔한 실수

**하나, 한 fixture 클래스에 관련 없는 도메인을 다 몰아넣기.** 도메인 하나당 fixture 클래스 하나. (`UserFixture`, `OrderFixture` 분리)

**둘, 팩토리 메서드에 파라미터를 너무 많이 받게 만들기.** 파라미터가 많아지면 그건 이미 fixture가 아니라 그냥 생성자다. 케이스별로 메서드를 나눠라 (`activeAllocation()`, `deletedAllocation()`처럼).

**셋, 테스트 검증에서 쓸 상수를 안 빼두기.** fixture 메서드 안에 값을 숨겨두면, 나중에 `assertThat(result).isEqualTo(???)` 할 때 다시 하드코딩하게 된다. 상수로 빼서 양쪽에서 같이 쓴다.

## 어디에 두나

**보통 `src/test/.../도메인/fixture/도메인Fixture.java`.** 프로덕션 코드와 같은 패키지 구조를 테스트 쪽에 그대로 따라가되, `fixture` 하위 폴더에 둔다.

## 픽스처 vs 빌더 vs mock

셋 다 "테스트용 가짜"를 만드는데 역할이 다르다.

**픽스처(Object Mother 패턴)** — 완성된 상태의 객체를 이름으로 바로 꺼내 쓴다. `activeAllocation()`처럼 파라미터 없이 호출해서 "자주 쓰는 케이스"를 즉시 얻는다. 대신 세부 값을 조금씩 바꾸고 싶을 때는 유연하지 않다.

**Test Data Builder** — 필드를 하나씩 바꿔가며 조립한다.

```java
User user = UserTestBuilder.builder()
        .email("custom@test.com")
        .age(30)
        .build();
```

기본값은 빌더가 채워주고, 테스트가 신경 쓰는 필드만 명시적으로 오버라이드한다. 케이스 조합이 많아질수록(나이·상태·권한 등 여러 축이 섞일 때) 픽스처보다 이쪽이 낫다. 픽스처와 함께 써도 된다 — 빌더의 기본값 프리셋을 픽스처 상수로 채우는 식.

**Mockito `@Mock`** — 이건 아예 다른 것이다. 픽스처는 **실제 도메인 객체**(진짜 `new`, 진짜 상태), mock은 **가짜 협력 객체**(행동을 미리 정해준 인형)다. `DummyPortAllocation` 같은 엔티티/값 객체는 픽스처로 만들고, `DummyPortOpenStackService` 같은 외부 연동 인터페이스는 mock으로 대체한다. 엔티티까지 mock 처리하면 실제 검증 로직(`isActiveFor()` 같은)이 하나도 실행되지 않아 테스트가 텅 비게 된다.

## 픽스처가 커지면 생기는 문제

**하나, 메서드가 서로 의존해서 연쇄적으로 깨진다.** `activeAllocation()`이 `creatingAllocation()`을 부르고, 그게 `lbVipRequest()`를 부르는 식으로 체인이 길어지면, 맨 아래 메서드 하나 바꿨을 때 위에서 뭐가 깨지는지 추적하기 어려워진다. 체인은 3단 이내로 유지한다.

**둘, "이 필드가 왜 이 값인지" 아무도 모른다.** `RESOURCE_ID = "101"`처럼 의미 없는 상수가 쌓이면, 테스트가 실패했을 때 그 값이 왜 중요한지 fixture만 봐선 알 수 없다. 특정 케이스를 위해 의도적으로 고른 값이면 상수 이름에 그 의도를 담는다 (`DUPLICATE_RESOURCE_ID`처럼).

**셋, 모든 테스트가 같은 픽스처 인스턴스를 공유해서 오염된다.** `public static final User USER = ...`처럼 상수로 객체 자체를 공유하면, 한 테스트가 그 객체의 상태를 바꿨을 때 다른 테스트에 영향을 준다. 객체는 항상 메서드 호출로 매번 새로 만들어서 리턴한다 (`public static User basicUser() { return ... }`), 필드에 담아두지 않는다.

## 언제 픽스처를 안 쓰는 게 나은가

**테스트 케이스마다 값이 거의 다 다르면 픽스처가 오히려 방해된다.** 그럴 땐 그냥 테스트 메서드 안에서 직접 만드는 게 더 읽기 쉽다. 픽스처는 "여러 테스트가 반복해서 쓰는 표준 케이스"에만 쓴다.
