# 3주차 search 복합 인덱스 설계 근거

대상 쿼리는 다음 조건으로 신청을 검색한다.

```sql
WHERE status = ?
  AND enrolled_at BETWEEN ? AND ?
  AND fee >= ?
ORDER BY enrolled_at DESC
LIMIT 50
```

비교할 후보는 다음 두 개다.

```sql
(status, enrolled_at DESC, fee)
(status, fee, enrolled_at DESC)
```

## 1. 복합 인덱스의 정렬 방식

B-tree 복합 인덱스는 각 컬럼을 따로 정렬하지 않는다. 선언된 컬럼을 왼쪽부터 묶은 튜플 전체를 사전식으로 정렬한다.

`(status, fee, enrolled_at)`은 개념적으로 다음 순서다.

```text
CONFIRMED, 50000, 2026-08-31
CONFIRMED, 50000, 2026-08-20
CONFIRMED, 55000, 2026-08-30
CONFIRMED, 55000, 2026-08-10
CONFIRMED, 60000, 2026-08-31
```

`status = 'CONFIRMED'`는 하나의 값으로 고정되므로 해당 구간으로 바로 이동할 수 있다. 그러나 `fee >= 50000`은 여러 fee 값을 포함하는 범위다. 이 범위에는 fee별로 정렬된 여러 `enrolled_at` 묶음이 들어 있다.

fee 50,000 묶음 안에서는 날짜순이어도 다음 fee 묶음으로 넘어가면 날짜가 다시 시작한다. 따라서 여러 fee 구간을 합친 전체 결과는 `enrolled_at` 순서가 아니며, `ORDER BY enrolled_at DESC`를 위해 별도 정렬이 필요할 가능성이 높다.

## 2. 범위 조건 뒤 컬럼의 의미

“범위 조건을 만나면 뒤 컬럼은 인덱스를 사용하지 못한다”는 설명은 정확하지 않다.

첫 번째 범위 조건 뒤 컬럼은 일반적으로 B-tree에서 읽을 시작점과 끝점을 더 좁히는 연속된 탐색 범위로 사용하기 어렵다는 의미다. 뒤 컬럼도 MySQL의 실행 계획에 따라 다음 용도로 쓰일 수 있다.

- 인덱스에 저장된 값으로 조건을 거르는 Index Condition Pushdown
- 필요한 컬럼이 모두 인덱스에 있을 때 원본 테이블 접근을 줄이는 covering index
- 스캔 후 추가 필터

따라서 `(status, fee, enrolled_at)`에서는 다음과 같이 동작할 가능성이 높다.

```text
status = ?              탐색 구간 고정
fee >= ?                탐색 시작점 설정
enrolled_at BETWEEN ?   fee 범위를 읽는 동안 추가 필터
```

`enrolled_at`을 전혀 사용하지 않는 것이 아니라, 날짜 조건으로 인덱스 스캔의 연속된 시작·종료 범위를 충분히 좁히기 어렵다는 뜻이다.

## 3. 선택 후보의 컬럼별 역할

```sql
CREATE INDEX idx_enrollment_status_enrolled_at_fee
    ON study_enrollment (status, enrolled_at DESC, fee);
```

| 인덱스 컬럼 | 쿼리 조건 | 역할 |
| --- | --- | --- |
| `status` | `status = 'CONFIRMED'` | 등치 조건으로 한 status 구간을 고정한다. |
| `enrolled_at DESC` | `BETWEEN from AND to` | 고정된 status 안에서 날짜 범위의 시작점과 끝점을 정한다. |
| `fee` | `fee >= 50000` | 날짜 범위를 읽는 동안 추가 필터로 검사한다. |

기대하는 처리 순서는 다음과 같다.

```text
status 구간 고정
→ 한 달 날짜 범위만 연속 스캔
→ fee 조건 검사
→ 이미 최신순인 결과에서 조건에 맞는 50건 반환
```

목록 쿼리는 조건에 맞는 앞의 50건을 찾으면 읽기를 일찍 멈출 수 있다. COUNT 쿼리는 조건에 필요한 세 컬럼이 모두 인덱스에 있으므로 실행 계획에 따라 원본 테이블 접근을 줄이는 covering index 효과도 기대할 수 있다.

`DESC`는 쿼리 의도를 DDL에 명확히 나타낸다. MySQL 8은 단일 방향 정렬에서 ASC 인덱스를 역방향으로 읽을 수도 있으므로, 핵심은 DESC 자체보다 `status` 다음에 `enrolled_at`을 배치한 것이다.

## 4. 현재 데이터 분포에 따른 예상 범위

시드 데이터의 분포는 다음과 같다.

- 전체 신청: 200,000건
- status: 3종에 거의 균등
- enrolled_at: 최근 365일에 거의 균등
- fee: 10,000원부터 105,000원까지 5,000원 단위 20종에 거의 균등

현재 측정 조건의 예상 결과는 다음과 같다.

```text
status = CONFIRMED                 200,000 × 1/3 = 약 66,667건
8월 한 달                         66,667 × 31/365 = 약 5,662건
fee >= 50,000                     5,662 × 12/20 = 약 3,397건
실제 API count                                          3,460건
```

`(status, enrolled_at, fee)`는 약 5,600건의 날짜 범위를 읽으면서 fee를 거를 것으로 예상된다.

`(status, fee, enrolled_at)`이 날짜 조건으로 범위를 추가 축소하지 못한다면 예상 후보는 다음과 같다.

```text
200,000 × status 1/3 × fee 12/20 = 약 40,000건
```

이 후보는 약 40,000건을 읽고 별도 날짜 정렬까지 수행할 가능성이 있다. 따라서 현재의 “한 달을 검색해 최신 50건 반환” 패턴에서는 `(status, enrolled_at, fee)`가 우선 후보다.

## 5. 결론과 검증 기준

현재 데이터 분포와 요청 조건에서는 다음 인덱스를 최초 후보로 선택한다.

```sql
CREATE INDEX idx_enrollment_status_enrolled_at_fee
    ON study_enrollment (status, enrolled_at DESC, fee);
```

선택 근거는 다음 세 가지다.

1. 등치 조건인 `status`를 먼저 고정한다.
2. 선택도가 높은 한 달 날짜 범위를 두 번째 컬럼으로 좁힌다.
3. 목록 결과를 `enrolled_at DESC` 순서로 읽어 별도 정렬을 줄인다.

이 결론은 최초 가설이다. 인덱스 적용 후 동일 조건의 EXPLAIN에서 다음을 확인해 최종 판단한다.

- `Table scan`이 `Index range scan`으로 바뀌는가
- 예상 스캔 행 수가 약 20만 건에서 충분히 감소하는가
- 별도 `Sort` 단계가 제거되는가
- 목록 SELECT와 COUNT SELECT 모두 새 인덱스를 사용하는가
- 동일 요청의 실제 응답 시간이 감소하는가

날짜 범위가 대부분의 데이터를 포함하거나 fee 조건이 훨씬 더 선택적인 실제 조회 패턴이라면 `(status, fee, enrolled_at)`이 더 나을 수 있다. 그 경우 두 후보를 같은 조건으로 비교해 최종 순서를 바꾼다.
