# Type Inference (타입 추론)

TypeScript가 명시적인 타입 선언 없이 값이나 표현식으로부터 타입을 자동으로 결정하는 기능.

JavaScript에는 없는 개념이다. 선언만 해도 타입 정보가 생긴다.

## 기본 추론

```typescript
const name = 'jiho';     // string으로 추론
const count = 0;         // number로 추론
const active = true;     // boolean으로 추론
const list = [1, 2, 3]; // number[]로 추론
```

변수 초기화 시점에 타입이 결정된다. 이후 다른 타입의 값을 할당하면 에러.

```typescript
let count = 0;
count = 'hello'; // 에러: string을 number에 할당 불가
```

## 함수 반환 타입 추론

반환 타입을 명시하지 않아도 추론된다.

```typescript
function add(a: number, b: number) {
  return a + b; // 반환 타입: number (자동 추론)
}

async function fetchUser(id: string) {
  const user = await db.findUser(id);
  return user; // 반환 타입: Promise<User>
}
```

파라미터는 추론이 안 되므로 명시해야 한다. 반환 타입은 추론에 맡겨도 되지만, 공개 API나 복잡한 함수는 명시하는 게 가독성에 좋다.

## const vs let 추론 차이

```typescript
const str = 'hello';  // 타입: 'hello' (리터럴 타입)
let str2 = 'hello';   // 타입: string

const num = 42;       // 타입: 42 (리터럴 타입)
let num2 = 42;        // 타입: number
```

`const`는 재할당 불가이므로 더 좁은 리터럴 타입으로 추론한다. 이를 이용해 상수를 정밀하게 타이핑할 수 있다.

## as const

객체나 배열을 완전한 상수로 만들어 리터럴 타입을 유지한다.

```typescript
const config = {
  port: 3000,
  host: 'localhost',
} as const;
// { readonly port: 3000; readonly host: 'localhost'; }
// port의 타입이 number가 아니라 3000

const DIRECTIONS = ['left', 'right', 'up', 'down'] as const;
// readonly ['left', 'right', 'up', 'down']
type Direction = typeof DIRECTIONS[number]; // 'left' | 'right' | 'up' | 'down'
```

`as const`로 선언한 배열에서 union 타입을 파생시키는 패턴은 자주 쓰인다. 배열에 값을 추가하면 union 타입도 자동으로 확장된다.

## typeof 연산자 (타입 공간)

JavaScript의 `typeof`는 런타임에서 동작하지만, TypeScript에서는 타입 공간에서도 쓸 수 있다.

```typescript
const user = { id: '1', name: 'jiho' };
type User = typeof user; // { id: string; name: string; }

function createUser(name: string) { return { id: crypto.randomUUID(), name }; }
type UserResult = ReturnType<typeof createUser>; // { id: string; name: string; }
```

클래스나 함수에서 타입을 뽑아쓸 때 유용하다. 타입을 따로 선언하지 않고 구현체에서 파생시킬 수 있다.

## 추론 한계

TypeScript가 추론을 못하거나 부정확하게 하는 경우:
- 빈 배열: `const arr = []` → `never[]`로 추론됨. 타입 명시 필요
- JSON.parse 결과: 항상 `any`
- 외부 API 응답: 타입 보장 없음

이런 경우에는 명시적 타입 선언이나 Narrowing이 필요하다.

참고: narrowing.md, generics.md
