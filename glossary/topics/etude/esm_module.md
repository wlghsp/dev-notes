# ESM 모듈 시스템

## 한 줄 요약

Node.js에서 파일을 가져오는 방식. CommonJS(require)의 후속 표준이 ESM(import).

---

## CommonJS vs ESM

```javascript
// CommonJS (구방식)
const Docker = require('dockerode')
module.exports = { handleTerminal }

// ESM (현재 표준)
import Docker from 'dockerode'
export function handleTerminal() { ... }
```

이 프로젝트는 ESM을 씁니다. `package.json`에 `"type": "module"`이 있으면 ESM 모드입니다.

---

## Java import와 비교

```java
// Java
import com.github.dockerjava.api.DockerClient;
```

```typescript
// TypeScript ESM
import Docker from 'dockerode'
import type { WebSocket } from 'ws'  // 타입만 import (런타임엔 포함 안 됨)
```

`import type`은 TypeScript 전용으로, 타입 정보만 가져오고 컴파일 후 사라집니다. 런타임 오버헤드 없음.

---

## 로컬 파일 import 시 .js 필수

ESM에서 로컬 파일을 import할 때 반드시 `.js` 확장자를 붙여야 합니다.

```typescript
// 틀림
import { handleTerminal } from './terminal'

// 맞음
import { handleTerminal } from './terminal.js'
```

TypeScript 파일(`.ts`)이지만 컴파일되면 `.js`가 되므로 이렇게 씁니다. 처음엔 이상하게 느껴지지만 ESM 규칙입니다.

---

## 왜 tsconfig에 module: nodenext인가

```json
{
  "module": "nodenext",
  "moduleResolution": "nodenext"
}
```

Node.js의 ESM을 완전히 지원하는 설정입니다. `commonjs`로 하면 `require` 방식으로 컴파일되고, `nodenext`로 하면 `import` 방식으로 컴파일됩니다.

`package.json`의 `"type": "module"`과 세트로 써야 합니다. 둘 중 하나만 있으면 에러납니다.

---

## 자주 나오는 에러

| 에러 | 원인 | 해결 |
|------|------|------|
| `Cannot use import statement` | `type:module` 없음 | `package.json`에 `"type": "module"` 추가 |
| `ERR_UNKNOWN_FILE_EXTENSION .ts` | `ts-node` ESM 미지원 | `tsx`로 교체 |
| `Module not found` | `.js` 확장자 누락 | 로컬 import에 `.js` 붙이기 |
