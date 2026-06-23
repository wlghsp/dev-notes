# Fastify TypeScript Patterns

Fastify + TypeScript 조합에서 자주 나오는 패턴 모음. Java 백엔드 경험 기준으로 "이건 다르다" 싶은 것들 위주.

## 요청 타입 정의

Fastify는 요청 객체를 제네릭으로 타이핑한다.

```typescript
import { FastifyRequest, FastifyReply, RouteHandler } from 'fastify';

interface CreateUserBody {
  name: string;
  email: string;
}

interface UserParams {
  id: string;
}

interface UserQuery {
  page?: number;
  limit?: number;
}

// 핸들러에 타입 직접 선언
async function createUserHandler(
  request: FastifyRequest<{ Body: CreateUserBody }>,
  reply: FastifyReply
) {
  const { name, email } = request.body; // 타입 안전
  // ...
}

// 조합
async function getUserHandler(
  request: FastifyRequest<{ Params: UserParams; Querystring: UserQuery }>,
  reply: FastifyReply
) {
  const { id } = request.params;
  const { page = 1, limit = 20 } = request.query;
}
```

## RouteHandler 타입

핸들러 함수를 별도로 정의할 때 `RouteHandler` 타입을 쓰면 깔끔하다.

```typescript
import { RouteHandler } from 'fastify';

type CreateUserHandler = RouteHandler<{ Body: CreateUserBody }>;

const createUser: CreateUserHandler = async (request, reply) => {
  // request.body가 CreateUserBody로 추론됨
};
```

## JSON Schema 연동

Fastify는 JSON Schema로 요청 검증을 한다. TypeScript 타입과 별도로 관리해야 하는 게 단점이다. `@sinclair/typebox`나 `zod`로 스키마와 타입을 동시에 생성하는 방법으로 해결한다.

**TypeBox 방식**
```typescript
import { Type, Static } from '@sinclair/typebox';

const CreateUserSchema = Type.Object({
  name: Type.String(),
  email: Type.String({ format: 'email' }),
});

type CreateUserBody = Static<typeof CreateUserSchema>; // 타입 자동 파생

fastify.post('/users', {
  schema: { body: CreateUserSchema }, // 검증 + 타입 동시에
}, async (request: FastifyRequest<{ Body: CreateUserBody }>, reply) => {
  const { name, email } = request.body;
});
```

## Plugin 타입 (fp-ts/fastify-plugin)

`fastify-plugin`으로 플러그인 만들 때 타입 작성 방법.

```typescript
import fp from 'fastify-plugin';
import { FastifyPluginAsync } from 'fastify';

interface MyPluginOptions {
  prefix?: string;
}

const myPlugin: FastifyPluginAsync<MyPluginOptions> = async (fastify, options) => {
  fastify.decorate('myService', new MyService());
};

export default fp(myPlugin);
```

`fastify.decorate()`로 추가한 프로퍼티는 타입에 자동으로 반영되지 않는다. 별도로 타입 선언이 필요하다.

```typescript
// types.d.ts 또는 플러그인 파일 하단
declare module 'fastify' {
  interface FastifyInstance {
    myService: MyService;
  }
}
```

이 패턴을 **Module Augmentation**이라고 한다. 외부 모듈의 타입을 확장할 때 쓴다.

## 에러 타입

Fastify 에러 핸들러에서 에러 타입을 좁히는 방법.

```typescript
import { FastifyError } from 'fastify';

fastify.setErrorHandler((error, request, reply) => {
  if (error instanceof SomeCustomError) {
    reply.status(400).send({ message: error.message });
    return;
  }
  reply.status(500).send({ message: 'Internal Server Error' });
});
```

## 자주 쓰는 import

```typescript
import Fastify, {
  FastifyInstance,
  FastifyRequest,
  FastifyReply,
  FastifyPluginAsync,
  RouteHandler,
  FastifyError,
} from 'fastify';
```

## JavaScript 대비 달라지는 것들

JavaScript Fastify에서는 `request.body.name`처럼 그냥 접근하다가 런타임에 에러를 만났다면, TypeScript에서는 컴파일 타임에 잡힌다.

주요 차이:
- 핸들러 파라미터에 타입을 안 달면 `any`로 추론됨. 타입 에러가 안 나서 오히려 위험
- `request.body`는 기본적으로 `unknown`. 제네릭으로 타입을 명시해야 올바른 타입 제공
- `reply.send()`는 반환값 타입을 체크하지 않음. 응답 타입까지 강제하려면 추가 설정 필요

참고: generics.md, narrowing.md, type-inference.md
