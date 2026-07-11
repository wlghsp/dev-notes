# CommonJS (CJS)

Node.js가 채택한 JS 모듈 시스템. `require()`로 불러오고 `module.exports`로 내보낸다.

## 기본 형태

```javascript
// math.js
function add(a, b) {
  return a + b;
}
module.exports = { add };

// main.js
const { add } = require('./math');
console.log(add(1, 2));
```

## 핵심 특징 — 동기적, 런타임 로딩

CJS의 `require()`는 **실행 시점(runtime)에 동기적으로** 파일을 읽고 즉시 평가한다.

```javascript
if (condition) {
  const foo = require('./foo'); // 조건문 안에서도 자유롭게 호출 가능
}
```

이게 가능한 이유는 `require`가 그냥 함수 호출이기 때문이다. 문법적 제약이 없고, 코드가 실행되는 순서대로 그때그때 모듈을 읽어들인다.

## Node.js가 CJS를 쓴 이유

CJS는 2009년 Node.js 초창기에 표준화됐다. 당시 브라우저의 ES Module 표준(ESM)은 아직 존재하지 않았다. 서버 환경에서는 파일이 로컬 디스크에 있으므로 동기적으로 읽어도 성능 문제가 없었기 때문에 동기 로딩 방식이 자연스러운 선택이었다.

## ESM과의 근본적인 차이 — 정적 분석 가능 여부

`require()`는 런타임에 평가되는 함수 호출이라서, 번들러나 도구가 **코드를 실행하지 않고는** 어떤 모듈을 가져오는지 알 수 없는 경우가 있다.

```javascript
const moduleName = condition ? 'foo' : 'bar';
const mod = require(moduleName); // 정적 분석 불가능 — 실행해봐야 안다
```

반면 ESM의 `import`는 파일 최상단에만 올 수 있고 문자열 리터럴만 허용되는 문법적 제약이 있다. 이 제약 덕분에 번들러가 코드를 실행하지 않고도 의존성 그래프를 정확히 그릴 수 있다 (참고: esm.md).

이 차이가 tree shaking(사용하지 않는 코드 제거) 가능 여부를 가른다. CJS는 원칙적으로 tree shaking이 어렵고, ESM은 쉽다.

## exports 객체의 동작

```javascript
// module.exports는 하나의 객체(또는 값)를 통째로 내보낸다
module.exports = add;        // 함수 자체를 내보냄
module.exports.add = add;    // 프로퍼티로 추가
exports.add = add;           // module.exports와 같은 객체를 가리키는 축약 표현
```

`exports`와 `module.exports`는 처음엔 같은 객체를 가리키지만, `exports = {...}`처럼 재할당하면 연결이 끊어진다. 이 함정 때문에 보통 `module.exports`만 명시적으로 쓰는 게 관례다.

## Node.js가 모듈 시스템을 판단하는 방식

Node.js는 파일을 열 때 이걸 CJS로 해석할지 ESM으로 해석할지 결정해야 한다. 이 판단은 문법이 아니라 **확장자와 `package.json`의 `type` 필드**로 이루어진다.

- 확장자가 `.cjs` → 무조건 CJS로 해석
- 확장자가 `.mjs` → 무조건 ESM으로 해석
- 확장자가 `.js` → `package.json`의 `"type"` 필드를 따른다
  - `"type": "commonjs"` 또는 필드 없음(기본값) → CJS
  - `"type": "module"` → ESM

즉 `.js` 파일 하나만 봐서는 그게 CJS인지 ESM인지 파일 내용을 열어보기 전까지 알 수 없고, 같은 디렉토리의 `package.json` 설정에 의존한다.

## "masquerading as CJS/ESM" — Node.js가 실제로 쓰는 표현

Node.js 공식 문서와 에러 메시지에 등장하는 표현이다. **확장자와 `package.json`의 `type` 필드가 실제 문법과 어긋날 때** 발생하는 상황을 가리킨다.

예를 들어 `"type": "module"`인 패키지 안에 `import`/`export` 문법 없이 `require()`/`module.exports`만 있는 `.js` 파일이 있으면, Node.js는 이 파일을 ESM으로 해석하려 시도하다가 실패한다. 반대로 `.cjs` 확장자를 가진 파일 안에 `import` 문이 있으면, "이 파일은 CJS로 취급되도록 되어 있는데 ESM 문법을 쓰고 있다"는 뜻에서 **ESM 코드가 CJS로 위장(masquerade)하고 있다**고 표현한다.

```
SyntaxError: Cannot use import statement outside a module
```

이런 에러가 바로 이 불일치에서 나온다. 원인은 대체로 다음 중 하나다.

- `package.json`에 `"type": "module"`을 지정하지 않았는데 `import`/`export`를 쓴 경우
- 반대로 `"type": "module"`인 프로젝트에서 서드파티 스크립트나 설정 파일(`webpack.config.js` 등)이 CJS 문법으로 작성된 경우 — 이때는 파일 확장자를 `.cjs`로 바꿔서 "나는 CJS다"라고 명시적으로 선언해 문제를 피한다

해결의 핵심은 **문법을 고치는 것**이 아니라 **Node.js에게 이 파일이 어떤 모듈 시스템인지 확장자/`type` 필드로 명확히 알려주는 것**이다.

참고: esm.md, bundler.md
