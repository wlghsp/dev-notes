# Utility Types

TypeScript 내장 제네릭 타입. 기존 타입을 변환해서 새 타입을 만들 때 사용한다.

## 자주 쓰는 것들

**Partial<T>**
모든 프로퍼티를 optional로 만든다. PATCH 요청 바디처럼 일부만 업데이트할 때 유용하다.

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

type UpdateUserBody = Partial<User>;
// { id?: string; name?: string; email?: string; }
```

**Required<T>**
모든 프로퍼티를 required로 만든다. `Partial`의 반대.

**Pick<T, K>**
특정 프로퍼티만 골라서 새 타입을 만든다.

```typescript
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: string; name: string; }
```

**Omit<T, K>**
특정 프로퍼티를 제외하고 새 타입을 만든다.

```typescript
type CreateUserBody = Omit<User, 'id'>; // id는 서버가 생성
// { name: string; email: string; }
```

**Record<K, V>**
키와 값 타입을 지정해서 객체 타입을 만든다.

```typescript
type RolePermissions = Record<string, boolean>;
// { [key: string]: boolean }

type UserMap = Record<string, User>;
// { [userId: string]: User }
```

**Readonly<T>**
모든 프로퍼티를 readonly로 만든다. 변경 불가 설정 객체 등에 사용.

```typescript
const config: Readonly<Config> = { port: 3000, host: 'localhost' };
config.port = 4000; // 에러
```

**ReturnType<T>**
함수의 반환 타입을 추출한다.

```typescript
async function getUser() {
  return { id: '1', name: 'jiho' };
}

type UserResult = ReturnType<typeof getUser>; // Promise<{ id: string; name: string }>
type User = Awaited<ReturnType<typeof getUser>>; // { id: string; name: string }
```

**Awaited<T>**
Promise를 unwrap해서 resolve된 타입을 꺼낸다. async 함수 반환 타입 다룰 때 자주 씀.

## 조합 패턴

실제로는 이것들을 조합해서 쓴다.

```typescript
// DB 엔티티에서 API 요청 바디 타입 파생시키기
interface UserEntity {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}

type CreateUserRequest = Omit<UserEntity, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateUserRequest = Partial<Omit<UserEntity, 'id' | 'createdAt' | 'updatedAt'>>;
```

중복 타입 정의를 줄이는 게 핵심이다. 엔티티 하나에서 요청/응답 타입을 파생시키면 변경이 생겼을 때 한 곳만 수정하면 된다.

참고: generics.md, narrowing.md
