# npm run dev 내부 동작과 프로덕션 배포 구조

`npm run dev`로 개발 서버를 켤 때와, 실제 서버에 배포할 때 내부적으로 벌어지는 일은 완전히 다르다. 개발 환경은 빠른 피드백을 위한 구조이고, 배포 환경은 정적 파일을 최대한 효율적으로 서빙하는 구조다.

---

## `npm run dev` 실행 시 벌어지는 일

### 1. package.json의 script 실행

```json
{
  "scripts": {
    "dev": "vite"
  }
}
```

`npm run dev`는 사실 `package.json`에 정의된 `dev` 스크립트를 그대로 실행하는 것뿐이다. 이 예시에서는 `vite` 명령어를 실행한다.

### 2. Vite 개발 서버(로컬 HTTP 서버) 기동

Vite는 내부적으로 Node.js 기반 HTTP 서버를 띄운다. 이 서버는 진짜 웹서버(Nginx 같은)가 아니라, 개발 편의를 위해 만들어진 **미들웨어 서버**다. 기동 시 하는 일:

- `node_modules`의 의존성들을 esbuild로 사전 번들링 (참고: vite-why-fast.md)
- `vite.config.js`를 읽어 플러그인, alias, 프록시 설정 등을 로드
- `localhost:5173` 같은 포트에서 요청을 대기

### 3. 브라우저 요청이 들어왔을 때의 흐름

브라우저가 `http://localhost:5173`에 접속하면:

```
1. 브라우저 → GET / 요청
2. Vite 서버 → index.html 반환
   (이 html은 <script type="module" src="/src/main.jsx"> 를 포함)
3. 브라우저 → native ESM으로 /src/main.jsx 요청
4. Vite 서버 → main.jsx를 그때그때 변환해서 응답
   - JSX/TS 파일이면 esbuild로 즉시 트랜스파일
   - Vue 파일이면 컴파일러로 변환
   - 원본 그대로 번들링하지 않고 "요청된 파일 하나만" 처리
5. 브라우저가 main.jsx의 import 문을 보고 다음 모듈 요청
6. 4~5를 반복하며 필요한 모듈만 순차적으로 로드
```

즉 dev 서버는 미리 전체를 준비해두는 게 아니라, **브라우저가 요청하는 순간 해당 파일만 변환해서 돌려주는 온디맨드(on-demand) 방식**으로 동작한다. 이게 개발 서버 구동이 빠른 이유이기도 하다 (참고: vite-why-fast.md).

### 4. 파일 감시(watch)와 HMR

Vite는 파일 시스템을 감시하다가(fs.watch) 소스 파일이 바뀌면:

1. 바뀐 파일과 연관된 모듈 그래프의 일부만 무효화
2. 브라우저와 맺어둔 **WebSocket 연결**을 통해 "이 모듈이 바뀌었다"는 메시지 전송
3. 브라우저 쪽 Vite 클라이언트 런타임(`@vite/client`, HTML에 자동 주입됨)이 해당 모듈만 다시 import
4. 전체 페이지 새로고침 없이 화면 갱신 (React의 경우 컴포넌트 상태도 보존됨 — Fast Refresh)

정리하면 dev 서버는 **HTTP 서버 + 파일 변환기 + WebSocket 기반 실시간 통신 채널**을 겸하는 프로세스다.

---

## 프로덕션 배포 흐름 — build → dist → Nginx

### 1. 빌드 (`npm run build`)

```json
{
  "scripts": {
    "build": "vite build"
  }
}
```

이 단계에서는 dev 서버와 다르게 실제 번들링이 일어난다 (Rollup 기반, 참고: vite-why-fast.md).

- 모든 소스 코드를 정적 분석해서 의존성 그래프 완성
- tree shaking으로 사용하지 않는 코드 제거
- minify(공백/변수명 압축), 코드 분할(code splitting)
- CSS 추출 및 압축
- 파일명에 해시 추가 (`main.a1b2c3.js`) — 캐시 무효화 전략

결과물은 `dist/` 폴더에 생성된다.

```
dist/
├── index.html
├── assets/
│   ├── main.a1b2c3.js
│   ├── main.d4e5f6.css
│   └── logo.g7h8i9.svg
```

빌드가 끝나면 이 `dist/` 안의 파일들은 **순수 정적 파일**이다. React든 Vue든 상관없이 결과물은 그냥 HTML/CSS/JS/이미지 뭉치다. 더 이상 Node.js나 Vite가 필요 없다.

### 2. Nginx가 하는 일

Nginx는 이 정적 파일들을 그대로 파일시스템에서 읽어 클라이언트에게 응답하는 **정적 파일 서버(static file server)** 역할을 한다.

```nginx
server {
    listen 80;
    root /var/www/myapp/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

- `root`: `dist/` 폴더 경로를 가리킴
- `try_files $uri $uri/ /index.html`: 요청된 경로에 실제 파일이 없으면 `index.html`을 대신 반환 — SPA(Single Page Application)의 클라이언트 사이드 라우팅을 지원하기 위한 설정. `/users/123` 같은 경로로 직접 접속해도 Nginx가 그 경로에 해당하는 실제 파일을 찾는 대신 항상 `index.html`을 내려주고, 그 안의 JS 라우터(React Router 등)가 URL을 보고 알맞은 화면을 그린다

### 3. 왜 dev 서버 대신 Nginx를 쓰는가

| | Vite dev 서버 | Nginx |
|---|---|---|
| 목적 | 빠른 피드백, 파일 변환, HMR | 정적 파일을 빠르고 안정적으로 서빙 |
| 처리 방식 | 요청마다 변환(transform) 수행 | 디스크의 완성된 파일을 그대로 응답 |
| 성능 최적화 | 개발 편의 우선, 프로덕션 최적화 없음 | gzip/브로틀리 압축, 캐시 헤더, 정적 파일 서빙에 특화된 이벤트 기반 아키텍처 |
| 동시 접속 처리 | 소규모(개발자 1인) 가정 | 대량 동시 접속 처리 가능 |

Vite dev 서버는 Node.js 프로세스가 요청마다 파일을 읽고 변환하는 오버헤드가 있어서 대량 트래픽에 적합하지 않다. 반면 이미 완성된 정적 파일을 서빙하는 데는 Nginx 같은 C로 작성된 이벤트 기반 웹서버가 훨씬 효율적이다.

### 4. 전체 흐름 요약

```
[개발]
소스 코드 (.jsx, .ts, .css)
   ↓ npm run dev
Vite 개발 서버 (Node.js, 온디맨드 변환 + HMR)
   ↓ 브라우저에서 확인하며 작업

[배포]
소스 코드
   ↓ npm run build (Rollup 번들링, tree shaking, minify)
dist/ (정적 HTML/CSS/JS 결과물)
   ↓ 서버에 업로드 (scp, CI/CD 등)
Nginx가 dist/를 root로 지정해 정적 파일 서빙
   ↓
사용자 브라우저
```

빌드 이후 단계에서는 Vite, React, Vue 같은 도구/프레임워크의 존재가 완전히 사라진다는 점이 핵심이다. Nginx 입장에서는 그냥 HTML/CSS/JS 파일일 뿐이다.

참고: vite-why-fast.md, bundler.md, esm.md
