# Idempotency (멱등성)

같은 작업을 여러 번 실행해도 결과가 동일한 성질.

수학에서 온 개념이다. f(f(x)) = f(x)가 성립하면 멱등 함수다.

## 멱등한 것과 멱등하지 않은 것

멱등한 작업:
- `SET counter = 10` — 몇 번 실행해도 counter는 10
- `DELETE FROM orders WHERE id = 5` — 이미 없으면 아무 일도 안 일어남
- `PUT /users/1` (전체 교체) — 같은 데이터로 덮어쓰면 결과 같음

멱등하지 않은 작업:
- `counter += 1` — 실행할 때마다 값이 달라짐
- `INSERT INTO orders VALUES (...)` — 실행할 때마다 row가 추가됨
- `POST /payments` — 실행할 때마다 결제가 발생함

## 왜 중요한가

At-Least-Once 환경에서는 중복 전달이 일어날 수 있다. 이때 수신 측이 멱등하면 중복이 들어와도 결과가 달라지지 않는다. 멱등성이 보장된 시스템은 재시도가 자유롭다.

반대로 멱등하지 않은 시스템에서 재시도가 일어나면 부작용이 쌓인다. 이 문제를 해결하려면 deduplication을 추가하거나, 작업 자체를 멱등하게 설계해야 한다.

## 멱등하게 만드는 방법

**고유 키 기반 INSERT**
INSERT를 실행할 때 고유한 idempotency key를 함께 보낸다. 이미 같은 키로 처리된 기록이 있으면 무시한다. 결제 API에서 많이 쓰는 패턴이다.

**UPSERT**
`INSERT ... ON DUPLICATE KEY UPDATE` 또는 `INSERT ... ON CONFLICT DO NOTHING` 방식으로, 있으면 덮어쓰고 없으면 삽입한다. 중복 실행해도 결과가 같다.

**상태를 절대값으로 표현**
`+= 1` 대신 `= 현재값 + 1`처럼 현재 상태를 읽고 절대값으로 쓰는 방식은 멱등하지 않다. 진짜 멱등하게 만들려면 목표 상태를 명시해야 한다. `SET stock = 99`처럼.

참고: at-least-once.md
참고: deduplication.md
참고: exactly-once.md
