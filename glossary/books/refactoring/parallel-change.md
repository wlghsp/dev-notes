# Parallel Change (Expand and Contract)

참고: Refactoring 2nd Edition, Change Function Declaration(124)의 "migration mechanics"

## 문제

기존에 쓰이던 함수나 클래스의 시그니처, 또는 구조 자체를 바꿔야 하는데, 호출부가 여러 곳에 퍼져 있는 상황을 생각해보자. 한 번에 다 바꾸면 그 순간부터 컴파일이 깨지거나 테스트가 실패하는 상태를 지나야 한다. 리팩토링의 원칙은 "매 단계마다 동작하는 상태를 유지하며 작은 걸음으로 나아가는 것"인데, 구조를 한 번에 갈아엎으면 이 원칙이 깨진다.

## 해법: Expand → Migrate → Contract

기존 것을 고치는 대신, 새 버전을 나란히 두고 호출부를 하나씩 옮긴 뒤 다 옮기고 나서야 옛것을 걷어낸다.

1. **Expand** — 기존 함수/클래스는 그대로 둔 채 새 버전을 추가한다. 이 시점엔 기존 동작에 아무 영향이 없다.
2. **Migrate** — 기존 것을 호출하는 곳을 하나씩 찾아 새 버전 호출로 바꾼다. 한 곳 옮길 때마다 테스트를 돌려 확인할 수 있어서, 중간에 "테스트가 실패하는 상태"가 없다.
3. **Contract** — 모든 호출부가 옮겨지면 기존 함수/클래스를 삭제한다.

## 왜 이 이름인가

Expand(확장)와 Contract(수축) 사이에 낀 Migrate 단계 때문에 신구 버전이 나란히(parallel) 존재하는 기간이 생긴다. 그 기간 동안은 어느 시점에 봐도 테스트가 항상 초록불이어야 한다는 게 이 기법의 핵심 제약이다.

## Change Function Declaration과의 관계

책에서는 이 절차가 독립 기법이 아니라 Change Function Declaration(124)의 마이그레이션 실행 방법(migration mechanics)으로 등장한다. 함수 시그니처(이름, 인자)를 안전하게 바꾸고 싶을 때 쓰는 절차다. 새 시그니처의 함수를 만들고, 기존 함수 본문에서 새 함수를 호출하도록 위임한 뒤, 호출부를 하나씩 새 함수로 옮기고, 마지막에 기존 함수를 지운다.

## 코드로 보는 절차: 파라미터 추가하기 (책 원문 예시, p.130)

책은 도서 대출 시스템의 `addReservation`에 우선순위 큐 파라미터를 추가하는 예시로 이 절차를 보여준다. 호출부를 한 번에 찾아 바꾸기 어려운 상황(폴리모픽 메서드이거나 호출부가 많을 때)을 가정한다.

**시작 코드**

```javascript
class Book…

  addReservation(customer) {
    this._reservations.push(customer);
  }
```

**1단계 — Expand: Extract Function으로 새 함수를 만든다.** 아직 `addReservation`이라는 이름을 그대로 쓸 수 없으니(같은 이름의 두 함수가 공존할 수 없다) 검색하기 쉬운 임시 이름을 쓴다.

```javascript
class Book…

  addReservation(customer) {
    this.zz_addReservation(customer);
  }

  zz_addReservation(customer) {
    this._reservations.push(customer);
  }
```

**2단계 — 새 함수에 파라미터를 추가한다.**

```javascript
class Book…

  addReservation(customer) {
    this.zz_addReservation(customer, false);
  }

  zz_addReservation(customer, isPriority) {
    this._reservations.push(customer);
  }
```

책은 이 시점에 Introduce Assertion(302)을 끼워 넣어, 호출부를 옮길 때 새 파라미터를 빠뜨리면 바로 알아채도록 안전장치를 추가하라고 권한다.

```javascript
  zz_addReservation(customer, isPriority) {
    assert(isPriority === true || isPriority === false);
    this._reservations.push(customer);
  }
```

**3단계 — Migrate: Inline Function으로 호출부를 하나씩 옮긴다.** 옛 함수(`addReservation`)를 호출하던 곳에 Inline Function(115)을 적용하면, 그 호출부가 새 함수(`zz_addReservation`)를 직접 호출하는 형태로 바뀐다. 호출부가 여럿이면 한 곳씩 바꾸고 테스트하면서 진행할 수 있다.

**4단계 — Contract: 모든 호출부를 옮긴 뒤, 새 함수 이름을 원래 이름으로 되돌린다.** 이번엔 Change Function Declaration의 단순 절차(simple mechanics)만으로 충분하다 — 이름을 바꿀 대상은 이제 새 함수 하나뿐이고, 옛 함수는 이미 걷어냈기 때문이다.

```javascript
class Book…

  addReservation(customer, isPriority) {
    assert(isPriority === true || isPriority === false);
    this._reservations.push(customer);
  }
```

## 공개 API에서의 활용

책은 이 절차가 "직접 고칠 수 없는 코드가 호출하는" 공개 API를 바꿀 때 특히 유용하다고 말한다. 새 함수(`circumference` 같은)를 만든 시점에서 리팩토링을 잠시 멈추고, 옛 함수(`circum`)를 deprecated로 표시한 뒤 호출자들이 새 함수로 옮겨갈 때까지 기다릴 수 있다. 모두 옮겨간 게 확인되면(또는 영영 확인되지 않더라도) 그때 옛 함수를 지운다. 지호님이 원래 질문한 "domain 객체를 전 영역에서 쓰다가 영역별로 나누는" 상황이 바로 이 케이스다 — 호출부가 여러 영역에 흩어져 있어서 한 번에 못 바꾸는 상황.

## 적용 사례: domain 객체를 영역별로 분리하기

domain 객체 하나가 여러 영역(예: 주문 영역, 배송 영역)에 걸쳐 쓰이고 있을 때 이 패턴이 그대로 적용된다. 영역별 새 클래스를 먼저 만들고(Expand), 호출부를 영역 하나씩 새 클래스로 옮기고(Migrate), 다 옮긴 뒤 원래 domain 객체에서 그 영역 몫을 걷어낸다(Contract). 이 작업의 실행 단계에서는 extract-class.md(책임을 분리해 새 클래스를 뽑는 기법)와 move-function.md, move-field.md(뽑아낸 클래스로 실제 코드를 옮기는 기법)를 함께 쓰게 된다.

## 데이터베이스에서의 같은 패턴

책 2장(Problems with Refactoring, Databases 절)에서 파울러는 이 절차를 데이터베이스 필드(컬럼) 리네이밍 사례로 직접 "parallel change(또는 expand-contract)"라 부르며 언급한다. 새 필드를 추가만 하고 아직 쓰지 않는 커밋 → 업데이트 로직이 옛 필드와 새 필드를 동시에 갱신 → 읽는 쪽을 점진적으로 새 필드로 이전 → 모두 옮겨간 게 확인되고 버그가 드러날 시간을 충분히 준 뒤 옛 필드 제거, 순서다. 코드 버전과 다른 점은, 데이터베이스에서는 실제 운영 데이터가 이전할 시간을 벌기 위해 이 단계들이 여러 번의 릴리스에 걸쳐 나뉜다는 것이다(problems-with-refactoring.md).

## 관련 기법

change-function-declaration.md — 이 마이그레이션 절차가 원래 소속된 상위 기법.
extract-class.md — domain 객체 분리에서 새 영역별 클래스를 만드는 단계.
move-function.md, move-field.md — 호출부를 새 클래스로 실제로 옮기는 실행 기법.
problems-with-refactoring.md — 이 절차가 데이터베이스 마이그레이션에도 그대로 쓰이는 사례.
