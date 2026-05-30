# Full-Text Search (전문 검색)

문서 전체 내용에서 특정 단어나 표현을 찾는 검색 방식. 정확한 값 매칭이 아니라 관련성(relevance)을 기준으로 결과를 반환한다.

## 정확한 매칭과의 차이

정확한 매칭(Exact Match): "category = '전자기기'" 처럼 값이 완전히 일치해야 한다. DB의 WHERE 절과 같다.

전문 검색(Full-Text Search): "노트북 추천"을 검색했을 때 "노트북을 강력 추천합니다"라는 문장도 찾아낸다. 단어 단위로 분석하고 관련성이 높은 순서로 정렬한다.

## 동작 원리

1. 색인 시: Analyzer가 텍스트를 토큰으로 분리해 역색인에 저장.
2. 검색 시: 쿼리 텍스트도 같은 Analyzer로 토큰화.
3. 토큰 단위로 역색인을 조회해 매칭 문서를 찾는다.
4. 각 문서의 relevance score를 계산해 점수 높은 순으로 반환.

## Elasticsearch에서

`match` 쿼리가 전문 검색의 기본이다.

```json
GET /products/_search
{
  "query": {
    "match": {
      "description": "노트북 추천"
    }
  }
}
```

"노트북 추천"을 토큰화하면 ["노트북", "추천"]. 두 토큰 중 하나라도 포함된 문서를 찾고, 더 많이 포함될수록 점수가 높다.

## `text` 필드에만 적용

전문 검색은 Analyzer를 거치는 `text` 타입 필드에서만 의미가 있다. `keyword` 필드는 분석을 하지 않으므로 전문 검색 대상이 아니다.

참고: inverted-index.md, analyzer.md, relevance-score.md, query-dsl.md, field.md
