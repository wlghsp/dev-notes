# Field (필드)

Document를 구성하는 키-값 쌍. 관계형 DB의 column에 해당한다.

## 타입

Elasticsearch의 필드는 타입을 가진다.

- `text`: 전문 검색(full-text search)용. Analyzer를 거쳐 토큰으로 분리돼 역색인에 저장된다.
- `keyword`: 정확한 값 매칭, 정렬, 집계용. 분석 없이 원문 그대로 저장된다.
- `integer`, `long`, `float`: 숫자
- `date`: 날짜
- `boolean`: true/false
- `object`, `nested`: 중첩 JSON 구조

## text vs keyword

같은 문자열 데이터라도 용도에 따라 타입이 달라진다.

"노트북"이라는 상품명을 저장할 때:
- "노트북으로 검색 가능해야 한다" → `text`
- "정확히 '노트북'인 것만 필터링한다" → `keyword`

실무에서는 하나의 필드를 `text`와 `keyword` 두 타입으로 동시에 저장하는 멀티 필드(multi-field) 방식을 자주 쓴다.

## 동적 매핑

문서를 색인할 때 Mapping에 없는 필드가 나타나면 Elasticsearch가 타입을 추론해 자동으로 추가한다. 문자열은 기본적으로 `text` + `keyword` 멀티 필드로 매핑된다.

참고: document.md, mapping.md, analyzer.md
