# Relevance Score (관련성 점수)

검색 결과가 쿼리와 얼마나 관련있는지를 나타내는 점수. 점수가 높을수록 결과 상위에 노출된다.

## BM25

Elasticsearch 5.0부터 기본 점수 계산 알고리즘으로 BM25를 사용한다.

참고: bm25.md

## 점수에 영향을 주는 요소

TF (Term Frequency): 검색 단어가 문서에 많이 등장할수록 점수가 높다. 단, BM25는 등장 횟수가 일정 이상이면 점수 증가가 둔해진다 — 단어 도배로 순위를 올리는 것을 방지한다.

IDF (Inverse Document Frequency): 검색 단어가 전체 문서에서 희귀할수록 점수가 높다. "은", "는", "이", "가" 같은 흔한 단어는 점수 기여가 낮다.

Field Length: 짧은 필드에서 단어가 나타날수록 점수가 높다. 10단어짜리 문서에서 "노트북"이 등장하는 것이 1000단어짜리 문서에서 등장하는 것보다 더 관련성이 높다고 판단한다.

## 점수 확인

```json
GET /products/_search
{
  "explain": true,
  "query": {
    "match": { "title": "노트북" }
  }
}
```

`explain: true`를 붙이면 각 문서의 점수 계산 과정을 상세히 볼 수 있다.

## Filter Context는 점수 없음

`filter` 절 안의 조건은 yes/no만 판단하고 score에 영향을 주지 않는다. 정렬이나 점수가 필요 없는 조건은 filter로 처리해야 성능상 유리하다.

참고: bm25.md, full-text-search.md, query-dsl.md, inverted-index.md
