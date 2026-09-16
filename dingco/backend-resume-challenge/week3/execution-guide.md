# 3주차 실행 가이드

`week3/prep-questions.md`로 준비한 뒤, `studypass`에서 **신청 검색의 실행 계획을 개선하고 회원별 집계를 사전 집계로 바꾸는** 순서다. 대상 저장소는 `/Users/jihochoi/Documents/study/dingco/challenge-backend-resume-2026-08-wlghsp-r17`이다.

2주차와 달리 이번 주차의 핵심 산출물은 HTTP 부하 수치만이 아니다. **동일 데이터·동일 파라미터의 개선 전/후 EXPLAIN**과 그 실행 시간을 반드시 남긴다. 실제로 나온 결과만 아래 코드 블록에 붙여 넣는다.

---

## 1단계: 브랜치와 측정 조건 고정

홈페이지에서 `submit/week-03__weekly-pr` 브랜치를 생성한다.

- [ ] 브랜치 생성 완료
- [ ] `application.yml`의 시드 설정을 기록했다.

기본 시드는 다음이며, 이번 측정 전후에 바꾸지 않는다.

```text
members=2000 / studies=300 / enrollments=200000 / commentsPerStudy=30
```

검색 파라미터도 전후에 완전히 같아야 한다. 시더는 실행 시점 기준 최근 365일에 신청을 균등하게 생성하고 status를 3종류로 균등하게 섞는다. 따라서 아래처럼 **한 달 범위 + CONFIRMED + 중간 이상의 fee**를 고정하면, 인덱스 차이가 드러날 가능성이 높다. 날짜는 실행 당일에 맞춰 최근 365일 안으로 조정한다.

```bash
export SEARCH_FROM='2026-08-01T00:00:00'
export SEARCH_TO='2026-08-31T23:59:59'
export SEARCH_STATUS='CONFIRMED'
export SEARCH_MIN_FEE='50000'
export STATS_MIN_FEE='50000'
```

위 예시 날짜가 시드 범위 밖이면 결과가 0건이므로 사용하면 안 된다. 먼저 아래 SQL에서 `min(enrolled_at)`, `max(enrolled_at)`을 보고 실제 범위 안의 한 달을 선택한다.

## 2단계: 깨끗한 개선 전 DB와 앱 기동

`schema.sql`은 기존 테이블을 변경하지 않는다. 이전 실습 DB가 있으면 새 DDL을 추가해도 자동 반영되지 않는다. **개선 전 측정은 인덱스가 없는 깨끗한 DB**에서 시작한다.

현재 DB를 지워도 되는 경우에만 다음을 실행한다. `down -v`는 MySQL 볼륨의 기존 로컬 데이터를 지우므로, 보존할 데이터가 있다면 백업하거나 별도 DB를 쓴다.

```bash
cd /Users/jihochoi/Documents/study/dingco/challenge-backend-resume-2026-08-wlghsp-r17
docker compose down -v
docker compose up -d
./gradlew bootRun
```

첫 기동은 20만 건 시드 때문에 시간이 걸린다. 로그에서 기동 완료를 확인한 뒤 다음을 실행한다.

```bash
docker compose exec db mysql -ustudypass -pstudypass1234 studypass -e \
  "SELECT COUNT(*) AS enrollments, MIN(enrolled_at) AS min_enrolled_at, MAX(enrolled_at) AS max_enrolled_at FROM study_enrollment; SHOW INDEX FROM study_enrollment;"
```

### 2-1. 개선 전 데이터·인덱스 확인 결과

```text
+-------------+----------------------------+----------------------------+
| enrollments | min_enrolled_at            | max_enrolled_at            |
+-------------+----------------------------+----------------------------+
|      200000 | 2025-09-15 04:36:24.309847 | 2026-09-15 04:27:24.309847 |
+-------------+----------------------------+----------------------------+
+------------------+------------+----------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table            | Non_unique | Key_name             | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+------------------+------------+----------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| study_enrollment |          0 | PRIMARY              |            1 | id          | A         |      199580 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| study_enrollment |          1 | fk_enrollment_study  |            1 | study_id    | A         |         300 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| study_enrollment |          1 | fk_enrollment_member |            1 | member_id   | A         |        1980 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+------------------+------------+----------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
```

기대 상태는 `PRIMARY`만 존재하는 것이다. `study_id`, `member_id`가 외래 키여도 MySQL이 해당 FK 컬럼에 인덱스를 자동으로 만들 수 있으므로, `SHOW INDEX` 결과를 기준으로 쓰고 “PK 외 인덱스가 전혀 없다”고 단정하지 않는다. 이번 검색 조건(`status`, `enrolled_at`, `fee`)용 인덱스가 없다는 사실이 중요하다.

## 3단계: 개선 전 search를 재현하고 측정

새 터미널에서 앱이 떠 있는 상태로 실행한다.

```bash
curl -sS -G 'http://localhost:8080/api/enrollments/search' \
  --data-urlencode "from=$SEARCH_FROM" \
  --data-urlencode "to=$SEARCH_TO" \
  --data-urlencode "status=$SEARCH_STATUS" \
  --data-urlencode "minFee=$SEARCH_MIN_FEE" | jq '{count, first: .content[0]}'





# 워밍업 3회 후, 같은 요청을 5회 측정한다.
for i in 1 2 3; do
  curl -sS -o /dev/null -G 'http://localhost:8080/api/enrollments/search' \
    --data-urlencode "from=$SEARCH_FROM" --data-urlencode "to=$SEARCH_TO" \
    --data-urlencode "status=$SEARCH_STATUS" --data-urlencode "minFee=$SEARCH_MIN_FEE"
done
for i in 1 2 3 4 5; do
  curl -sS -o /dev/null -w "search before #$i %{time_total}s\\n" -G 'http://localhost:8080/api/enrollments/search' \
    --data-urlencode "from=$SEARCH_FROM" --data-urlencode "to=$SEARCH_TO" \
    --data-urlencode "status=$SEARCH_STATUS" --data-urlencode "minFee=$SEARCH_MIN_FEE"
done
```

### 3-1. 개선 전 search 응답과 5회 측정

```text
{
  "count": 3460,
  "first": {
    "fee": 95000,
    "id": 169261,
    "status": "CONFIRMED",
    "enrolledAt": "2026-08-31T23:57:24.309847"
  }
}

search before #1 0.130773s
search before #2 0.118893s
search before #3 0.116113s
search before #4 0.117484s
search before #5 0.117067s
```

`search`는 목록 SELECT와 `countSearch` COUNT SELECT를 모두 수행한다. 둘을 따로 EXPLAIN해야 API 비용을 빠뜨리지 않는다.

```bash
docker compose exec db mysql -ustudypass -pstudypass1234 studypass -e "
EXPLAIN FORMAT=TREE
SELECT id, study_id, member_id, status, fee, enrolled_at, created_at
FROM study_enrollment
WHERE enrolled_at BETWEEN '$SEARCH_FROM' AND '$SEARCH_TO'
  AND status = '$SEARCH_STATUS'
  AND fee >= $SEARCH_MIN_FEE
ORDER BY enrolled_at DESC
LIMIT 50;

EXPLAIN FORMAT=TREE
SELECT COUNT(*)
FROM study_enrollment
WHERE enrolled_at BETWEEN '$SEARCH_FROM' AND '$SEARCH_TO'
  AND status = '$SEARCH_STATUS'
  AND fee >= $SEARCH_MIN_FEE;"
```

### 3-2. 개선 전 search EXPLAIN

```text
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Limit: 50 row(s)  (cost=20207 rows=50)
    -> Sort: study_enrollment.enrolled_at DESC, limit input to 50 row(s) per chunk  (cost=20207 rows=199580)
        -> Filter: ((study_enrollment.enrolled_at between '2026-08-01T00:00:00' and '2026-08-31T23:59:59') and (study_enrollment.`status` = 'CONFIRMED') and (study_enrollment.fee >= 50000))  (cost=20207 rows=199580)
            -> Table scan on study_enrollment  (cost=20207 rows=199580)
 |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                                                                                                                           |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Aggregate: count(0)  (cost=20281 rows=1)
    -> Filter: ((study_enrollment.enrolled_at between '2026-08-01T00:00:00' and '2026-08-31T23:59:59') and (study_enrollment.`status` = 'CONFIRMED') and (study_enrollment.fee >= 50000))  (cost=20207 rows=739)
        -> Table scan on study_enrollment  (cost=20207 rows=199580)
 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

표 형식 EXPLAIN을 썼다면 최소한 `type`, `possible_keys`, `key`, `rows`, `Extra`를 남긴다. `type=ALL`, `key=NULL`, `rows`가 전체 행 수에 가까우면 풀 스캔 근거다. `ORDER BY` 때문에 `Using filesort`가 보이는지도 기록한다.

## 4단계: search 복합 인덱스 설계·적용

이번 API의 조건은 다음 순서다.

```sql
WHERE status = ?
  AND enrolled_at BETWEEN ? AND ?
  AND fee >= ?
ORDER BY enrolled_at DESC
```

첫 구현 후보는 다음이다.

```sql
CREATE INDEX idx_enrollment_status_enrolled_at_fee
    ON study_enrollment (status, enrolled_at DESC, fee);
```

### 4-1. 복합 인덱스가 정렬되는 방식

B-tree 복합 인덱스는 각 컬럼을 독립적으로 정렬해 두는 구조가 아니다. 선언된 컬럼을 왼쪽부터 묶은 **튜플 전체를 사전식으로 정렬**한다.

예를 들어 `(status, fee, enrolled_at)` 인덱스의 일부는 개념적으로 다음 순서다.

```text
CONFIRMED, 50000, 2026-08-31
CONFIRMED, 50000, 2026-08-20
CONFIRMED, 55000, 2026-08-30
CONFIRMED, 55000, 2026-08-10
CONFIRMED, 60000, 2026-08-31
...
```

`status = 'CONFIRMED'`는 하나의 값으로 고정되므로 그 구간으로 바로 이동할 수 있다. 하지만 다음 조건인 `fee >= 50000`은 하나의 값이 아니라 50,000부터 끝까지 이어지는 **범위**다. 이 범위 안에는 fee별로 정렬된 여러 `enrolled_at` 묶음이 존재한다.

따라서 `(status, fee, enrolled_at)`에서 `enrolled_at`을 전혀 사용하지 못한다는 뜻은 아니다. MySQL은 인덱스에 들어 있는 `enrolled_at` 값을 필터 검사에 사용할 수 있다. 정확한 의미는 다음과 같다.

> 첫 번째 범위 조건인 `fee >= 50000` 뒤의 `enrolled_at`은 B-tree에서 읽을 시작점과 끝점을 더 좁히는 연속된 탐색 범위로 사용하기 어렵다.

또한 fee가 50,000인 묶음 안에서는 날짜순이어도, 그 다음 fee 55,000 묶음으로 넘어가면 날짜가 다시 시작한다. 여러 fee 구간을 합친 전체 결과는 `enrolled_at` 순서가 아니다. 그래서 다음 정렬을 별도로 수행해야 할 가능성이 높다.

```sql
ORDER BY enrolled_at DESC
```

이를 간단히 표현하면 다음과 같다.

```text
(status, fee, enrolled_at)
          └─ 첫 범위 조건
                     └─ 필터에는 쓸 수 있지만 탐색 범위와 전체 날짜 정렬에는 불리함
```

### 4-2. `(status, enrolled_at, fee)`를 우선 후보로 둔 이유

선택한 인덱스는 다음 순서로 정렬된다.

```text
CONFIRMED, 2026-08-31, 50000
CONFIRMED, 2026-08-31, 60000
CONFIRMED, 2026-08-30, 55000
CONFIRMED, 2026-08-29, 95000
...
```

쿼리 조건과 연결하면 각 컬럼의 역할은 다음과 같다.

| 인덱스 컬럼 | 쿼리 조건 | 역할 |
| --- | --- | --- |
| `status` | `status = 'CONFIRMED'` | 등치 조건으로 한 status 구간을 고정한다. |
| `enrolled_at DESC` | `BETWEEN from AND to` | 고정된 status 안에서 날짜 범위의 시작점과 끝점을 정한다. |
| `fee` | `fee >= 50000` | 날짜 범위를 읽는 동안 추가 필터로 검사한다. |

이 순서의 장점은 다음과 같다.

1. `status`를 등치 조건으로 먼저 고정한다.
2. 그 안에서 `enrolled_at`의 한 달 범위만 연속적으로 읽는다.
3. 인덱스가 이미 `enrolled_at DESC` 순서이므로 목록 쿼리의 정렬과 방향이 같다.
4. 읽는 도중 `fee >= 50000`을 검사하고, 조건을 만족하는 앞의 50건을 찾으면 목록 조회를 멈출 수 있다.
5. COUNT 쿼리에 필요한 세 조건이 모두 인덱스에 있으므로, 실행 계획에 따라 원본 테이블 접근을 줄이는 covering index 효과도 기대할 수 있다.

`DESC` 지정은 쿼리 의도를 DDL에 명확하게 표현한다. MySQL 8은 단일 방향 정렬이라면 ASC 인덱스를 역방향으로 읽을 수도 있으므로, `DESC` 자체보다 `status` 다음에 `enrolled_at`을 배치한 것이 핵심이다.

### 4-3. 현재 데이터 분포로 비교한 후보별 예상 범위

시더의 데이터는 다음과 같이 만들어진다.

- 전체 신청: 200,000건
- status: `CONFIRMED`, `CANCELLED`, `PENDING` 3종에 거의 균등
- enrolled_at: 최근 365일에 거의 균등
- fee: 10,000원부터 105,000원까지 5,000원 단위 20종에 거의 균등

현재 측정 조건을 적용하면 대략 다음과 같이 예상할 수 있다.

```text
status = CONFIRMED                         약 200,000 × 1/3 = 66,667건
8월 한 달                                약 66,667 × 31/365 = 5,662건
fee >= 50,000 (20개 값 중 12개)           약 5,662 × 12/20 = 3,397건
실제 API count                                                   3,460건
```

따라서 `(status, enrolled_at, fee)`는 대략 status와 한 달 범위에 해당하는 약 5,600건을 연속 스캔하면서 fee를 거르는 형태를 기대할 수 있다.

반면 `(status, fee, enrolled_at)`은 날짜가 인덱스 탐색 범위를 충분히 줄이지 못하면 다음 범위를 먼저 읽게 된다.

```text
200,000 × status 1/3 × fee 12/20 ≈ 40,000건
```

그리고 이 약 40,000건은 fee별 날짜순 묶음이므로 전체 `enrolled_at DESC` 결과를 만들기 위한 추가 정렬도 필요할 가능성이 높다. 현재의 “한 달 검색 후 최신 50건” 패턴에서는 `(status, enrolled_at, fee)`가 더 유리할 것이라는 가설을 세울 수 있다.

### 4-4. 범위 조건 뒤 컬럼에 대한 정확한 표현

“범위 조건을 만나면 그 뒤 컬럼은 인덱스를 사용하지 못한다”라고 단정하면 부정확하다. MySQL 8에서는 뒤 컬럼도 다음과 같이 쓰일 수 있다.

- 인덱스에 저장된 값만 보고 조건을 거르는 Index Condition Pushdown
- 쿼리에 필요한 컬럼이 모두 인덱스에 있을 때 원본 테이블 접근을 줄이는 covering index
- 옵티마이저가 선택하는 다른 접근 방식

다만 뒤 컬럼은 일반적으로 B-tree의 **연속된 시작·종료 탐색 범위**를 추가로 좁히는 데 제약이 생긴다. 이번 설계에서는 이 차이를 다음처럼 표현한다.

```text
status = ?              탐색 범위 고정에 사용
enrolled_at BETWEEN ?   탐색 시작점·끝점에 사용
fee >= ?                스캔 중 추가 필터에 사용
```

이것이 `CREATE INDEX idx_enrollment_status_enrolled_at_fee ON study_enrollment (status, enrolled_at DESC, fee)`를 최초 후보로 정한 근거다.

이것은 “정답”이 아니라 현재 시드 분포·고정한 파라미터를 전제로 한 최초 가설이다. EXPLAIN에서 날짜 범위가 지나치게 넓거나 `fee`가 훨씬 더 선택적이면, 두 후보를 실제로 비교하고 선택 이유를 evidence에 적는다. `status`는 3값 균등 분포라 단독 인덱스로는 후보에서 제외한다.

`src/main/resources/schema.sql`의 `study_enrollment` CREATE TABLE 뒤에 위 DDL을 추가한다. 이미 떠 있는 DB에는 `schema.sql` 수정만으로 반영되지 않으므로, 아래 중 하나를 선택한다.

```bash
# 개발 DB에 즉시 인덱스만 적용할 때
docker compose exec db mysql -ustudypass -pstudypass1234 studypass -e \
  "CREATE INDEX idx_enrollment_status_enrolled_at_fee ON study_enrollment (status, enrolled_at DESC, fee);"

# schema.sql부터 다시 검증할 때: 2단계의 down -v → up -d → bootRun을 다시 수행
```

### 4-5. 적용한 DDL과 선택 근거

```sql
index idx_enrollment_enrolled_at_fee(status, enrolled_at desc, fee),

status는 등치조건(=)이므로 첫 번째 컬럼으로 두어 조회 범위를 먼저 고정했다.
enrolled_at은 한 달 범위 조건이면서 ORDER BY enrolled_at DESC에 사용되므로 두 번째로 배치했다. 이를 통해 날짜 범위를 인덱스로 탐색하고 별도 정렬 비용을 줄이고자 했다.
fee도 범위 조건(>=)이므로 세 번째에 배치해 날짜 범위를 읽는 과정에서 추가 필터로 사용하도록 했다.

대안인 (status, fee, enrolled_at)은 fee >= 범위 뒤에서 날짜 조건으로 탐색 범위를 좁히기 어렵고, 전체 결과가 날짜순으로 정렬되지 않아 별도 정렬이 필요할 가능성이 있어 선택하지 않았다.
```

- [ ] `SHOW INDEX FROM study_enrollment`로 실제 생성 확인
- [ ] 대안 `(status, fee, enrolled_at)`을 선택하지 않은 이유를 EXPLAIN/분포와 연결해 기록

## 5단계: 같은 조건에서 search 재측정

2~3단계와 **동일한 환경, 시드, 파라미터, 워밍업 3회, 측정 5회**로 다시 실행한다.

```bash
for i in 1 2 3; do
  curl -sS -o /dev/null -G 'http://localhost:8080/api/enrollments/search' \
    --data-urlencode "from=$SEARCH_FROM" --data-urlencode "to=$SEARCH_TO" \
    --data-urlencode "status=$SEARCH_STATUS" --data-urlencode "minFee=$SEARCH_MIN_FEE"
done
for i in 1 2 3 4 5; do
  curl -sS -o /dev/null -w "search after #$i %{time_total}s\\n" -G 'http://localhost:8080/api/enrollments/search' \
    --data-urlencode "from=$SEARCH_FROM" --data-urlencode "to=$SEARCH_TO" \
    --data-urlencode "status=$SEARCH_STATUS" --data-urlencode "minFee=$SEARCH_MIN_FEE"
done
```

3단계의 두 EXPLAIN SQL도 그대로 다시 실행한다.

```
search after #1 0.031181s
search after #2 0.011043s
search after #3 0.009636s
search after #4 0.011664s
search after #5 0.009740s

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Limit: 50 row(s)  (cost=4776 rows=50)
    -> Index range scan on study_enrollment using idx_enrollment_status_enrolled_at_fee over (status = 'CONFIRMED' AND '2026-08-31 23:59:59.000000' <= enrolled_at <= '2026-08-01 00:00:00.000000' AND 50000 <= fee), with index condition: ((study_enrollment.enrolled_at between '2026-08-01T00:00:00' and '2026-08-31T23:59:59') and (study_enrollment.`status` = 'CONFIRMED') and (study_enrollment.fee >= 50000))  (cost=4776 rows=10612)
 |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Aggregate: count(0)  (cost=2608 rows=1)
    -> Filter: ((study_enrollment.enrolled_at between '2026-08-01T00:00:00' and '2026-08-31T23:59:59') and (study_enrollment.`status` = 'CONFIRMED') and (study_enrollment.fee >= 50000))  (cost=2254 rows=3537)
        -> Covering index range scan on study_enrollment using idx_enrollment_status_enrolled_at_fee over (status = 'CONFIRMED' AND '2026-08-31 23:59:59.000000' <= enrolled_at <= '2026-08-01 00:00:00.000000' AND 50000 <= fee)  (cost=2254 rows=10612)
 |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

### 5-1. search 전후 비교

| 항목 | 개선 전 | 개선 후 |
| --- | ---: | ---: |
| 목록 SELECT EXPLAIN의 key/type/rows | 인덱스 없음 / `Table scan` / 199,580행 | `idx_enrollment_status_enrolled_at_fee` / `Index range scan` / 10,612행 |
| COUNT SELECT EXPLAIN의 key/type/rows | 인덱스 없음 / `Table scan` / 199,580행 | `idx_enrollment_status_enrolled_at_fee` / `Covering index range scan` / 10,612행(필터 후 추정 3,537행) |
| API 5회 측정값 | 130.773 / 118.893 / 116.113 / 117.484 / 117.067ms | 31.181 / 11.043 / 9.636 / 11.664 / 9.740ms |
| API 평균 | 120.066ms | 14.653ms(약 87.8% 감소) |
| API 중앙값 | 117.484ms | 11.043ms(약 90.6% 감소) |

개선 후 목록 쿼리는 전체 테이블 스캔과 별도 Sort가 사라지고 복합 인덱스 범위 스캔으로 변경됐다. COUNT 쿼리는 검색 조건이 모두 포함된 covering index range scan을 사용한다. EXPLAIN의 `rows`는 실제 처리 행 수가 아닌 옵티마이저 추정치지만, 예상 스캔 범위가 199,580행에서 10,612행으로 줄었다. 동일 조건의 API 5회 측정에서는 중앙값이 117.484ms에서 11.043ms로 약 90.6% 감소했다.

목록 SELECT가 빨라져도 API 전체 시간은 `countSearch`에 지배될 수 있다. 그래서 목록과 count의 플랜을 분리해 해석한다. HTTP 시간만 개선됐다고 인덱스 효과라고 단정하지 않는다.

## 6단계: stats 개선 전 측정과 한계 확인

`stats`는 `fee >= ?`를 거른 뒤 회원별로 `COUNT/SUM/MAX`를 계산하고 `member_id`로 GROUP BY 및 count 내림차순 정렬한다. `minFee=0`이면 거의 전체 20만 건을 집계해야 하므로, search용 인덱스는 이 API를 충분히 해결하지 못할 수 있다.

```bash
curl -sS -G 'http://localhost:8080/api/enrollments/stats' \
  --data-urlencode "minFee=$STATS_MIN_FEE" | jq '.[0:3]'

for i in 1 2 3; do
  curl -sS -o /dev/null -G 'http://localhost:8080/api/enrollments/stats' --data-urlencode "minFee=$STATS_MIN_FEE"
done
for i in 1 2 3 4 5; do
  curl -sS -o /dev/null -w "stats before #$i %{time_total}s\\n" -G 'http://localhost:8080/api/enrollments/stats' \
    --data-urlencode "minFee=$STATS_MIN_FEE"
done

docker compose exec db mysql -ustudypass -pstudypass1234 studypass -e "
EXPLAIN FORMAT=TREE
SELECT member_id, COUNT(*) AS enrollment_count, SUM(fee) AS total_fee, MAX(enrolled_at) AS last_enrolled_at
FROM study_enrollment
WHERE fee >= $STATS_MIN_FEE
GROUP BY member_id
ORDER BY enrollment_count DESC;"
```

### 6-1. 개선 전 stats 측정·EXPLAIN

```text
재구성 측정: 집계 테이블 전환 전 API 응답 시간은 기록하지 못했으므로,
현재 동일 MySQL·동일 원본 데이터·동일 minFee=50000에서 기존 원본 집계 SQL을 EXPLAIN ANALYZE로 실행했다.

-> Limit: 50 row(s)  (actual time=228..228 rows=50 loops=1)
    -> Sort: enrollment_count DESC  (actual time=228..228 rows=50 loops=1)
        -> Group aggregate  (actual time=9.48..227 rows=2000 loops=1)
            -> Filter: fee >= 50000  (actual time=9.44..220 rows=120206 loops=1)
                -> Index scan on study_enrollment using fk_enrollment_member
                   (actual time=9.43..213 rows=200000 loops=1)
```

## 7단계: stats는 집계 테이블로 구현

이번 미션의 요구사항은 인덱스만으로 한계가 있으면 **집계 테이블 또는 반정규화 중 하나를 실제 적용**하는 것이다. 여기서는 원본 `study_enrollment`를 유지하고, 읽기 전용 요약을 분리하는 집계 테이블을 선택한다.

`minFee`가 임의의 값이라 회원당 단일 합계 행만 두면 조건을 정확히 재현할 수 없다. 이 시더의 fee는 10,000~105,000의 5,000 단위 20값이므로, 회원·fee별로 미리 집계한다. 최대 약 `2000 × 20 = 40000`행이라 원본 20만 행보다 훨씬 작고, 런타임에는 fee 조건을 걸고 다시 회원 단위로 합산할 수 있다.

### 7-1. 스키마

`schema.sql`에 추가할 후보다.

```sql
CREATE TABLE IF NOT EXISTS member_enrollment_fee_stats (
    member_id BIGINT NOT NULL,
    fee INT NOT NULL,
    enrollment_count BIGINT NOT NULL,
    total_fee BIGINT NOT NULL,
    last_enrolled_at DATETIME(6) NOT NULL,
    PRIMARY KEY (member_id, fee),
    INDEX idx_member_enrollment_fee_stats_fee_member (fee, member_id),
    CONSTRAINT fk_member_enrollment_fee_stats_member
        FOREIGN KEY (member_id) REFERENCES member (id)
);
```

### 7-2. 시드/기존 데이터 백필

시더가 `study_enrollment`을 JdbcTemplate으로 직접 삽입하므로, JPA 서비스만 고쳐서는 최초 데이터의 집계가 만들어지지 않는다. `SeedRunner`의 신청 데이터 배치 삽입 **직후** 다음 집계를 실행해 초기 데이터를 채운다.

기존 신청 데이터가 있는 경우에는 `member_enrollment_fee_stats`가 비어 있을 때만 최초 백필한다. 집계 데이터까지 이미 존재하면 앱을 시작할 때마다 20만 건을 다시 집계하지 않고 건너뛴다. 구체적인 분기 코드는 [`stats-implementation-guide.md`](stats-implementation-guide.md)에 있다.

```sql
INSERT INTO member_enrollment_fee_stats
    (member_id, fee, enrollment_count, total_fee, last_enrolled_at)
SELECT
    aggregated.member_id,
    aggregated.fee,
    aggregated.new_enrollment_count,
    aggregated.new_total_fee,
    aggregated.new_last_enrolled_at
FROM (
    SELECT
        member_id,
        fee,
        COUNT(*) AS new_enrollment_count,
        SUM(fee) AS new_total_fee,
        MAX(enrolled_at) AS new_last_enrolled_at
    FROM study_enrollment
    GROUP BY member_id, fee
) AS aggregated
ON DUPLICATE KEY UPDATE
    enrollment_count = aggregated.new_enrollment_count,
    total_fee = aggregated.new_total_fee,
    last_enrolled_at = aggregated.new_last_enrolled_at;
```

MySQL 8.0.20부터 `ON DUPLICATE KEY UPDATE` 안의 `VALUES(column)` 함수는 폐기 예정이다. `INSERT ... SELECT` 백필에서는 위처럼 집계 SELECT를 파생 테이블 `aggregated`로 감싸고 그 별칭을 참조하면 경고 없이 같은 UPSERT 동작을 수행한다.

### 7-3. 조회와 쓰기 정합성

- `MemberEnrollmentStatsRepository`에서 집계 테이블을 조회하고 신규 신청의 통계를 UPSERT한다.
- `EnrollmentController.stats()`는 기존 원본 집계 대신 새 repository를 호출한다.
- `EnrollmentService.enroll()`은 신청 저장과 집계 갱신을 같은 `@Transactional` 범위에서 실행한다.
- `SeedRunner`는 집계 테이블이 비어 있을 때만 기존 신청 또는 새 시드 데이터를 최초 백필한다.

파일별 전체 코드는 [`stats-implementation-guide.md`](stats-implementation-guide.md)에 분리했다.

현재 프로젝트에는 취소·수정 API가 없으므로 신규 신청 증가만 구현한다. 향후 취소·수정 기능이 생기면 count·total·last 값을 함께 보정해야 하며, 가장 최근 신청 삭제 시 `MAX(enrolled_at)`은 남은 원본에서 다시 계산해야 한다.

집계 테이블을 만들고 원본 집계를 유지하는 방식은 안 된다. controller가 실제로 새 조회를 호출하는지, 원본과 새 결과가 같은지 테스트로 확인한다.

## 8단계: stats 동일성·성능 회귀 테스트

`src/test/java/.../enrollment/EnrollmentControllerTest.java`를 새로 만든다. 최소 검증은 다음이다.

1. 서로 다른 `member`, `fee`, `enrolledAt`의 Enrollment를 저장한다.
2. `/api/enrollments/search`를 호출해 `count`와 `content`가 기대 조건/정렬과 일치하는지 검증한다.
3. 같은 원본 데이터에서 기존 집계 SQL(테스트에서 직접 계산한 기대값)과 `/api/enrollments/stats?minFee=...`의 회원별 count·totalFee·lastEnrolledAt이 같은지 검증한다.
4. 경계값: `fee == minFee`는 포함, 기간의 from/to 경계는 포함, 다른 status는 제외를 넣는다.

인덱스 자체를 단위 테스트로 검증하지 않는다. 인덱스는 DDL과 EXPLAIN으로, 기능 동일성은 API 회귀 테스트로 증명한다.

복사 가능한 `EnrollmentControllerTest` 전체 코드는 [`regression-test-guide.md`](regression-test-guide.md)에 있다. 테스트는 내부 repository 호출 결과가 아니라 `/api/enrollments/search`, `/api/enrollments/stats`의 HTTP 응답을 검증한다.

```bash
./gradlew test --tests '*Enrollment*Test'
./gradlew test
```

### 8-1. 테스트 결과

```text
BUILD SUCCESSFUL in 4s
5 actionable tasks: 1 executed, 4 up-to-date
```

## 9단계: stats 개선 후 재측정

6단계와 같은 `$STATS_MIN_FEE`, 워밍업 3회, 측정 5회로 다시 실행한다. 새 집계 테이블 대상 EXPLAIN도 남긴다.

```bash
for i in 1 2 3; do
  curl -sS -o /dev/null -G 'http://localhost:8080/api/enrollments/stats' --data-urlencode "minFee=$STATS_MIN_FEE"
done
for i in 1 2 3 4 5; do
  curl -sS -o /dev/null -w "stats after #$i %{time_total}s\\n" -G 'http://localhost:8080/api/enrollments/stats' \
    --data-urlencode "minFee=$STATS_MIN_FEE"
done
```

위 워밍업/5회 측정 전에 아래 명령으로 집계 테이블 크기와 API 응답을 확인하고, 측정 뒤 EXPLAIN을 실행한다.

```bash
# 집계 테이블의 행 수와 API 응답 확인
docker compose exec db mysql -ustudypass -pstudypass1234 studypass -e \
  "SELECT COUNT(*) AS stats_rows, COUNT(DISTINCT member_id) AS members FROM member_enrollment_fee_stats;"

curl -sS -G 'http://localhost:8080/api/enrollments/stats' \
  --data-urlencode "minFee=$STATS_MIN_FEE" | jq '.[0:3]'

# 새 집계 테이블을 읽는 실행 계획
docker compose exec db mysql -ustudypass -pstudypass1234 studypass -e "
EXPLAIN FORMAT=TREE
SELECT member_id,
       SUM(enrollment_count) AS enrollment_count,
       SUM(total_fee) AS total_fee,
       MAX(last_enrolled_at) AS last_enrolled_at
FROM member_enrollment_fee_stats
WHERE fee >= $STATS_MIN_FEE
GROUP BY member_id
ORDER BY enrollment_count DESC
LIMIT 50;"
```

### 9-1. 개선 후 stats 결과

#### 집계 테이블 행 수와 API 응답

```text
# stats_rows / members 결과와 curl 응답 앞 3건을 붙여 넣기
+------------+---------+
| stats_rows | members |
+------------+---------+
|      39732 |    2000 |
+------------+---------+
```

#### API 5회 측정 결과

```text
# stats after #1 ~ #5 결과를 붙여 넣기
stats after #1 0.041532s
stats after #2 0.010047s
stats after #3 0.010720s
stats after #4 0.011292s
stats after #5 0.010702s
```

#### 개선 후 EXPLAIN

```text
# EXPLAIN FORMAT=TREE 출력을 그대로 붙여 넣기
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Limit: 50 row(s)
    -> Sort: enrollment_count DESC, limit input to 50 row(s) per chunk
        -> Stream results  (cost=5585 rows=2040)
            -> Group aggregate: sum(member_enrollment_fee_stats.enrollment_count), sum(member_enrollment_fee_stats.total_fee), max(member_enrollment_fee_stats.last_enrolled_at)  (cost=5585 rows=2040)
                -> Filter: (member_enrollment_fee_stats.fee >= 50000)  (cost=3748 rows=18376)
                    -> Index scan on member_enrollment_fee_stats using PRIMARY  (cost=3748 rows=36753)
 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

재구성 측정(EXPLAIN ANALYZE, 동일 MySQL·동일 minFee=50000):
-> Limit: 50 row(s)  (actual time=31..31 rows=50 loops=1)
    -> Sort: enrollment_count DESC  (actual time=31..31 rows=50 loops=1)
        -> Group aggregate  (actual time=3.05..29.7 rows=2000 loops=1)
            -> Filter: fee >= 50000  (actual time=3.04..26.4 rows=23835 loops=1)
                -> Index scan on member_enrollment_fee_stats using PRIMARY
                   (actual time=3.03..24.7 rows=39732 loops=1)
```

`member_enrollment_fee_stats`의 행 수가 원본 `study_enrollment`의 200,000행보다 충분히 작은지 확인한다. EXPLAIN에서는 실제로 집계 테이블을 읽는지, `fee` 인덱스를 사용하는지, GROUP BY/ORDER BY 때문에 남는 정렬 또는 임시 테이블 비용이 무엇인지 기록한다.

| 지표 | 개선 전 | 개선 후 |
| --- | ---: | ---: |
| 읽는 테이블/대상 행 수 | `study_enrollment` / 실제 200,000행 스캔 | `member_enrollment_fee_stats` / 실제 39,732행 스캔 |
| EXPLAIN key/type/rows | `fk_enrollment_member` / `Index scan` / 실제 200,000행 (fee 필터 후 120,206행) | `PRIMARY` / `Index scan` / 실제 39,732행 (fee 필터 후 23,835행) |
| DB 쿼리 실제 완료 시간 (`EXPLAIN ANALYZE`) | 228ms | 31ms (약 86.4% 감소) |
| API 5회 측정값 | 확인 불가 — 전환 전 HTTP 측정 미기록 | 41.532 / 10.047 / 10.720 / 11.292 / 10.702ms |
| API 평균 | 미측정 — 6단계 5회 결과 필요 | 16.859ms |
| API 중앙값 | 미측정 — 6단계 5회 결과 필요 | 10.720ms |

개선 후에는 원본 200,000행 대신 39,732행의 집계 테이블을 읽는다. 다만 `fee`가 복합 PRIMARY KEY `(member_id, fee)`의 두 번째 컬럼이므로, 현재 EXPLAIN은 `idx_member_enrollment_fee_stats_fee_member`가 아니라 PRIMARY KEY 전체를 스캔한다. `ORDER BY enrollment_count DESC`도 집계 결과에 대한 Sort로 남아 있다. 즉 집계 테이블로 읽는 데이터량은 줄었지만, `fee` 조건과 정렬까지 모두 인덱스로 해결된 상태는 아니다.

6단계의 개선 전 API 5회 시간과 EXPLAIN을 채운 뒤에만 응답 시간 감소율을 계산한다. 현재 값만으로는 전후 성능 개선율을 주장하지 않는다.

## 10단계: resume·evidence·PR

`resume/resume.md`의 문장 3은 실제 측정값이 나온 뒤에만 채운다. 형식 예시는 다음이며 대괄호를 실제 값으로 바꾼다.

```text
20만 건 신청 데이터에서 EXPLAIN으로 [개선 전 플랜]을 확인한 뒤
(status, enrolled_at, fee) 복합 인덱스와 회원·금액별 사전 집계를 적용해,
동일 조건의 search [중앙값/평균]을 [전]ms에서 [후]ms로,
stats를 [전]ms에서 [후]ms로 개선했다.
```

`evidence/week-03__weekly-pr.md`에는 아래를 빠짐없이 쓴다.

- [ ] 시드 규모, OS/DB/JDK, 고정한 search/stats 파라미터, 워밍업·반복 횟수
- [ ] search 목록과 count 각각의 개선 전/후 EXPLAIN 원문
- [ ] search와 stats의 개선 전/후 5회 원시 시간 및 대표값
- [ ] 최종 인덱스 DDL과 `(status, enrolled_at, fee)` 순서의 근거, 버린 대안
- [ ] stats 집계 테이블의 스키마·백필·조회·쓰기 정합성 코드 경로
- [ ] 회귀 테스트 경로와 `./gradlew test` 통과 출력
- [ ] 근거형 질문 1~4의 최초 판단 → 연결한 코드/로그 → 검증 후 답변
- [ ] `## 리뷰 반영`: 최초에는 `자동 리뷰 수신 전`, 리뷰가 없으면 `지적 없음`, 있으면 지적과 보완 파일

마지막으로 제출 계약을 확인한다.

- [ ] `resume/resume.md`, `src/main/`, `src/test/`, `evidence/week-03__weekly-pr.md`를 모두 변경했다.
- [ ] `missions/`, `challenge.json`, 채점 workflow는 변경하지 않았다.
- [ ] 동일 결과 테스트와 실제 EXPLAIN/시간 증거가 저장소 안에 있다.

---

## 근거형 질문 1~4 답변 방향

1. **병목**: 개선 전 목록 SELECT와 COUNT의 `type/key/rows/Extra`를 근거로 쓴다. search와 stats를 한 문장으로 섞지 말고, search는 조건 탐색·정렬, stats는 대량 그룹 집계라는 서로 다른 비용을 구분한다.
2. **인덱스**: `(status, enrolled_at, fee)`를 선택했다면 `=` → 범위+정렬 → 후속 범위 순서와 실제 EXPLAIN을 근거로 쓴다. `fee`가 더 선택적이라는 실측이 나오면 그 결과에 맞춰 판단을 고친다.
3. **검증**: 같은 DB 시드·파라미터·워밍업·반복 횟수에서 EXPLAIN 전후와 HTTP 5회 원시값을 비교하고, API 회귀 테스트로 결과 동일성을 확인했다고 쓴다.
4. **쓰기 저하 대처**: 인덱스마다 INSERT/UPDATE 시 B-tree 유지 비용이 추가된다. 실제 쓰기 경로와 측정으로 필요한 인덱스만 남기고, 중복/저선택성 인덱스를 제거한다. 집계 테이블은 읽기를 빠르게 하는 대신 원본 쓰기 때 집계 갱신 비용과 정합성 관리가 추가된다는 트레이드오프를 함께 쓴다.
