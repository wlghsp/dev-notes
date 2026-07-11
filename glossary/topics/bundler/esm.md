# ES Modules (ESM)

JS 언어 표준 자체에 포함된 모듈 시스템. `import`/`export` 문법을 쓴다. ES2015(ES6)에서 도입됐고, 이후 브라우저와 Node.js 양쪽에 정착했다.

## 기본 형태

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from './math.js';
console.log(add(1, 2));
```

Node.js에서 쓰려면 `package.json`에 `"type": "module"`을 지정하거나 파일 확장자를 `.mjs`로 써야 한다.

## 핵심 특징 — 정적 구조

ESM의 `import`/`export`는 반드시 파일 최상단에만 올 수 있고, 조건문이나 함수 안에서 쓸 수 없다.

```javascript
if (condition) {
  import foo from './foo.js'; // 문법 에러 — import는 최상단에만 가능
}
```

이 제약은 불편해 보이지만 의도된 설계다. 코드를 **실행하지 않고도** 어떤 모듈이 어떤 모듈을 의존하는지 파싱만으로 알아낼 수 있게 만든다 (정적 분석, static analysis). 이것이 CJS의 `require()`와의 근본적 차이다 (참고: cjs.md).

## 정적 구조가 가능하게 하는 것들

**Tree shaking** — 번들러가 실제로 사용되는 export만 골라내고 나머지는 번들에서 제외할 수 있다. 실행 없이 의존성 그래프를 정확히 그릴 수 있기 때문에 가능하다.

**병렬 로딩** — 브라우저나 Node.js가 모듈 그래프를 미리 파싱해서, 서로 의존하지 않는 모듈들을 동시에 요청할 수 있다. CJS처럼 한 줄씩 실행하며 다음 require를 발견하는 방식과 다르다.

**Native ESM (브라우저)** — 최신 브라우저는 번들링 없이 `<script type="module">`만으로 `import`를 그대로 실행할 수 있다. 브라우저가 모듈 그래프를 스스로 파싱하고 필요한 파일들을 병렬로 fetch한다. 이 능력이 Vite의 개발 서버 구조의 기반이 된다 (참고: vite-why-fast.md).

## 동적 import — 예외적으로 허용되는 런타임 로딩

정적 제약만 있으면 "조건에 따라 필요할 때만 로드"하는 코드 분할(code splitting)이 불가능하다. 그래서 ESM은 함수 형태의 `import()`를 별도로 제공한다.

```javascript
if (condition) {
  const mod = await import('./foo.js'); // Promise를 반환, 런타임에 로드
}
```

정적 `import`와 달리 이건 어디서든 호출 가능하고 Promise를 반환한다. Vite나 Webpack의 code splitting은 이 문법을 분기점으로 인식해서 별도 청크(chunk)로 분리한다.

## CJS와 상호운용 시 주의점

Node.js 생태계에는 아직 CJS로 작성된 패키지가 많다. ESM에서 CJS를 불러오는 건 대체로 되지만, CJS에서 ESM을 `require()`로 불러오는 건 안 된다 (CJS는 동기, ESM은 기본적으로 비동기 로딩이라 구조적으로 안 맞음). 이 비대칭성이 마이그레이션을 어렵게 만드는 주요 원인이다.

참고: cjs.md, bundler.md, vite-why-fast.md
