# sandbox 테이블

## 한 줄 요약

퀘스트 세트마다 어떤 Docker 이미지를 쓸지를 DB에서 관리하는 테이블.
코드를 수정하지 않고 새 환경을 추가할 수 있게 한다.

---

## 왜 테이블로 관리하는가

처음에는 sandboxType에 따라 코드에서 이미지를 분기했다:

```typescript
// 나쁜 예 — 새 환경 추가할 때마다 코드 수정 필요
if (sandboxType === 'linux') image = 'ubuntu'
else if (sandboxType === 'docker') image = 'docker:dind'
```

sandbox 테이블로 옮기면 새 환경은 INSERT 한 줄로 끝난다. 코드는 그대로.

---

## 구조

```sql
CREATE TABLE sandbox (
  type        VARCHAR(20) PRIMARY KEY,  -- 식별자 (linux, linux-ssh, docker, ...)
  image       VARCHAR(100) NOT NULL,    -- Docker 이미지 이름
  binds       JSON,                     -- 볼륨 마운트 설정 (없으면 NULL)
  description TEXT                      -- 설명
);
```

현재 데이터:

| type | image | binds | 설명 |
|---|---|---|---|
| `linux` | `ubuntu` | NULL | 기본 리눅스 환경 |
| `linux-ssh` | `etude-ssh` | NULL | SSH 데몬 포함 |
| `docker` | `docker:dind` | NULL | Docker-in-Docker |
| `k8s` | `etude-k8s` | NULL | kubectl 포함 (향후) |

---

## 어떻게 쓰이는가

`quest_set` 테이블이 `sandbox_type`으로 sandbox를 FK 참조한다.

```
sandbox (type='linux', image='ubuntu')
    ↑ FK
quest_set (sandbox_type='linux')
```

터미널 연결 시 흐름:

```
1. 프론트가 sandboxType을 WebSocket URL에 포함
2. index.ts가 파라미터 파싱 → handleTerminal(sandboxType) 호출
3. sandbox.ts의 getSandboxConfig(sandboxType) — DB 조회
4. { image, binds } 반환 → createContainer에 사용
```

---

## getSandboxConfig

```typescript
// backend/src/sandbox.ts
export async function getSandboxConfig(sandboxType: string) {
    const [rows] = await db.query(
        'SELECT image, binds FROM sandbox WHERE type = ?',
        [sandboxType]
    )
    const row = rows[0]
    return {
        image: row.image,
        binds: typeof row.binds === 'string' ? JSON.parse(row.binds) : row.binds,
    }
}
```

`binds`를 JSON.parse하는 이유: MariaDB JSON 컬럼이 드라이버에 따라 문자열로 오는 경우가 있어서 명시적으로 파싱한다.

---

## 새 환경 추가 방법

코드 변경 없이 `init.sql`에 INSERT 한 줄만 추가하면 된다:

```sql
INSERT INTO sandbox (type, image, binds, description) VALUES
  ('k8s', 'etude-k8s', NULL, 'kubectl 포함 환경');
```

단, `terminal.ts`의 `SandboxType` union 타입에는 추가해야 한다:

```typescript
export type SandboxType = 'linux' | 'linux-ssh' | 'docker' | 'k8s'
```

docker 계열처럼 연결 방식이 다른 환경이라면 `handleTerminal`의 분기도 추가.
