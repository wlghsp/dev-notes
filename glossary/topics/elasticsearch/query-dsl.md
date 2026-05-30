# Query DSL

Elasticsearch에서 검색 쿼리를 JSON으로 표현하는 언어. SQL처럼 구조화된 문법을 JSON으로 구현한 것이다.

## 두 가지 컨텍스트

Query Context: "이 문서가 얼마나 관련있는가?" relevance score를 계산한다. `query` 절 안에 위치.

Filter Context: "이 조건을 만족하는가?" yes/no만 판단. score 계산 없음. 캐싱된다. `filter` 절 안에 위치.

필터로 처리할 수 있는 조건(날짜 범위, 카테고리 일치 등)은 `filter`에 넣는 게 성능상 유리하다.

## 주요 쿼리 타입

`match`: 전문 검색의 기본. 입력 텍스트를 분석해 토큰 단위로 검색한다.

```json
{ "match": { "title": "노트북 추천" } }
```

`term`: 정확한 값 매칭. 분석 없이 원문 그대로 비교. `keyword` 필드에 사용.

```json
{ "term": { "category": "전자기기" } }
```

`range`: 범위 조건.

```json
{ "range": { "price": { "gte": 100000, "lte": 500000 } } }
```

`bool`: 여러 쿼리를 논리적으로 조합. `must`(AND), `should`(OR), `must_not`(NOT), `filter`로 구성.

```json
{
  "bool": {
    "must": [{ "match": { "title": "노트북" } }],
    "filter": [{ "term": { "category": "전자기기" } }],
    "must_not": [{ "term": { "status": "품절" } }]
  }
}
```

`match_all`: 모든 문서 반환. score는 모두 1.0.

## bool 쿼리의 score 계산

`must`와 `should` 절은 score에 영향을 준다. `filter`와 `must_not`은 score에 영향을 주지 않는다.

`should`는 일치하지 않아도 되지만, 일치하면 score가 올라간다. `minimum_should_match`로 최소 일치 개수를 설정할 수 있다.

참고: full-text-search.md, relevance-score.md, field.md
