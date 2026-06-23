# Generics

타입을 파라미터로 받아 재사용 가능한 코드를 작성하는 기능. Java의 제네릭과 개념이 동일하다.

## 기본 형태

```typescript
function identity<T>(value: T): T {
  return value;
}

const result = identity<string>('hello'); // result: string
const result2 = identity(42);             // 타입 추론으로 T = number
```

`T`는 관례적인 이름일 뿐이다. `K`, `V`, `TItem` 등 맥락에 맞게 쓴다.

## 제약 조건 (Constraints)

`extends`로 타입 파라미터의 범위를 제한한다.

```typescript
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength('hello');  // OK
getLength([1, 2, 3]); // OK
getLength(42);       // 에러: number에는 length가 없음
```

Java의 `<T extends Comparable<T>>`와 동일한 개념이다.

## 인터페이스에서의 제네릭

```typescript
interface Repository<T> {
  findById(id: string): Promise<T>;
  save(entity: T): Promise<void>;
  findAll(): Promise<T[]>;
}

interface User {
  id: string;
  name: string;
}

class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User> { ... }
  async save(entity: User): Promise<void> { ... }
  async findAll(): Promise<User[]> { ... }
}
```

## 여러 타입 파라미터

```typescript
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

const p = pair('id', 123); // [string, number]
```

## Fastify에서의 제네릭

Fastify는 요청/응답 타입을 제네릭으로 받는다.

```typescript
import { FastifyRequest, FastifyReply } from 'fastify';

interface CreateUserBody {
  name: string;
  email: string;
}

async function createUser(
  request: FastifyRequest<{ Body: CreateUserBody }>,
  reply: FastifyReply
) {
  const { name, email } = request.body; // 타입 안전하게 접근
}
```

`FastifyRequest<{ Body: T }>` 패턴에서 `T`가 body의 타입이 된다. Params, Querystring도 같은 방식이다.

```typescript
FastifyRequest<{
  Params: { id: string };
  Querystring: { page: number };
  Body: CreateUserBody;
}>
```

참고: narrowing.md, utility-types.md, fastify-typescript-patterns.md
