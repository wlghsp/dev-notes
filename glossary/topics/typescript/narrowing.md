# Narrowing

TypeScript가 `union type`이나 `unknown` 타입을 더 구체적인 타입으로 좁혀가는 과정.

런타임 조건 분기를 보고 컴파일러가 "이 블록 안에서는 이 타입이 확실하다"는 것을 추론한다.

## 왜 필요한가

TypeScript는 정적 타입 언어지만, 실제 값은 런타임에 결정된다. `string | number`처럼 여러 타입이 될 수 있는 경우, 컴파일러 입장에서는 어느 쪽인지 알 수 없다. Narrowing은 개발자가 분기 조건을 작성하면 컴파일러가 그 의도를 이해하는 방식이다.

Java의 `instanceof` 체크 후 캐스팅과 비슷한 개념이지만, TypeScript에서는 캐스팅 없이 타입이 자동으로 좁혀진다.

## 주요 Narrowing 방법

**typeof 체크**
```typescript
function process(value: string | number) {
  if (typeof value === 'string') {
    // 이 블록 안에서 value는 string
    return value.toUpperCase();
  }
  // 여기서 value는 number
  return value.toFixed(2);
}
```

**instanceof 체크**
```typescript
function handle(error: Error | string) {
  if (error instanceof Error) {
    // Error 객체
    console.log(error.message);
  }
}
```

**in 연산자**
```typescript
type Cat = { meow: () => void };
type Dog = { bark: () => void };

function speak(animal: Cat | Dog) {
  if ('meow' in animal) {
    animal.meow(); // Cat으로 좁혀짐
  }
}
```

**타입 가드 함수**
```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

if (isString(input)) {
  // 여기서 input은 string
}
```

`value is string` 형태를 **type predicate**라고 부른다. 함수 반환 타입에 쓰며, true일 때 해당 타입으로 좁혀진다는 것을 컴파일러에 알린다.

## unknown vs any

`unknown`은 Narrowing을 강제하는 타입이다. `any`는 타입 체크를 포기하는 것이고, `unknown`은 "타입을 모르지만 사용하려면 먼저 좁혀라"는 의미다.

```typescript
function process(input: unknown) {
  input.toUpperCase(); // 에러: unknown에는 메서드 호출 불가
  
  if (typeof input === 'string') {
    input.toUpperCase(); // OK: string으로 좁혀짐
  }
}
```

외부 API 응답이나 JSON.parse 결과처럼 타입을 보장할 수 없는 값을 다룰 때 `unknown`을 쓰고 Narrowing으로 안전하게 처리한다.

## Exhaustive Check

`never` 타입을 이용해 모든 케이스를 처리했는지 컴파일 타임에 확인하는 패턴.

```typescript
type Shape = 'circle' | 'square' | 'triangle';

function getArea(shape: Shape): number {
  switch (shape) {
    case 'circle': return 3.14;
    case 'square': return 1;
    default:
      const _exhaustive: never = shape; // 'triangle'이 남으면 에러
      throw new Error(`Unhandled shape: ${_exhaustive}`);
  }
}
```

새로운 타입이 union에 추가됐을 때 처리를 빠뜨리면 컴파일 에러로 잡힌다.

참고: generics.md, utility-types.md
